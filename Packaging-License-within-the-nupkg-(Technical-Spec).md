# NuGet License expressions & embedded licenses technical design

* Status: **Implemented**
* Author(s): [Nikolche Kolev](https://github.com/nkolev92)
* Parent Spec: [Packaging License within the nupkg](https://github.com/NuGet/Home/wiki/Packaging-License-within-the-nupkg)

### SPDX specification

The SPDX specification provides a standardized approach to software licensing.
This specification talks about a the licenses format, and defines SPDX identifiers and expressions to allow authors to refer to a well-defined set of public licenses without the need to duplicate the said licenses files.
We will specifically target the way SPDX expressions and knowledge of the standard licenses are implemented into NuGet. As the above linked spec calls out, NuGet does not set any requirements about the content of the license files.
SPDX defines the existence of both Licenses and Exceptions, both of which are self-explanatory.
Both licenses and exceptions have a unique name, and most importantly a unique *case-sensitive* **identifier**.

Originally in version 1.0 of the SPDX specification, you could specify licenses by identifiers. As of the 2.0 version of the SPDX specification, more complex, SPDX expressions were defined.

* **License identifier**

The SPDX license identifier is defined [here](https://spdx.org/spdx-specification-21-web-version#h.4f1mdlm)
The license identifier syntax `1*(ALPHA / DIGIT / "-" / ".")`. 
The full [list](https://spdx.org/licenses/) of valid license identifiers.

* **[DocumentRef](https://spdx.org/spdx-specification-21-web-version#h.h430e9ypa0j9)**

Identify any external SPDX documents referenced within the SPDX document.
The syntax is `DocumentRef-1*(ALPHA / DIGIT / "-" / ".")`

* **[LicenseRef](https://spdx.org/spdx-specification-21-web-version#h.111kx3o)**

If a license is not on the SPDX license, this allows authors to specify their own license.
The syntax is `LicenseRef-1*(ALPHA / DIGIT / "-" / ".")`

* **[License operators](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60)**

The SPDX syntax defines 4 different operators.

- Disjunctive "OR" Operator
    * If presented with a choice between two or more licenses, use the disjunctive binary "OR" operator to construct a new license expression, where both the left and right operands are valid license expression values.

- Conjunctive "AND" Operator
    * If required to simultaneously comply with two or more licenses, use the conjunctive binary "AND" operator to construct a new license expression , where both the left and right operands are a valid license expression values.

- Exception "WITH" Operator
    * Sometimes a set of license terms apply except under special circumstances. In this case, use the binary "WITH" operator to construct a new license expression to represent the special exception situation.

- Unary "+" Operator
    * A single license is represented by using the short identifier from SPDX license list, optionally with a unary "+" operator following it to indicate "or later" versions may be applicable.


* **[SPDX Expression syntax](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60)**

The SPDX expression syntax allows authors to specify composite license with ease.
The ABNF of the syntax is as below:

References to appendix are from https://spdx.org/spdx-specification-21-web-version

```
idstring              = 1*(ALPHA / DIGIT / "-" / "." )

license-id            = <short form license identifier from https://spdx.org/spdx-specification-21-web-version#h.luq9dgcle9mo>

license-exception-id  = <short form license exception identifier from https://spdx.org/spdx-specification-21-web-version#h.ruv3yl8g6czd>

license-ref           = ["DocumentRef-"1*(idstring)":"]"LicenseRef-"1*(idstring)
 

simple-expression = license-id / license-id”+” / license-ref


compound-expression =  1*1(simple-expression /

                 simple-expression "WITH" license-exception-id /

                 compound-expression "AND" compound-expression /

                 compound-expression "OR" compound-expression ) /

                     "(" compound-expression ")" )


license-expression =  1*1(simple-expression / compound-expression)
```

Examples are:

* `(BSD-2-Clause OR MIT)`
* `Apache-2.0`

NuGet's approach to embedded licenses does not require package authors to conform to the SPDX specification. As per that assumption, NuGet will not be implementing the full SPDX license expression specification, but rather a subset of it.

### NuGet's License Expression ABNF

```
license-id            = <short form license identifier from https://spdx.org/spdx-specification-21-web-version#h.luq9dgcle9mo>

license-exception-id  = <short form license exception identifier from https://spdx.org/spdx-specification-21-web-version#h.ruv3yl8g6czd>

simple-expression = license-id / license-id”+”

compound-expression =  1*1(simple-expression /

                simple-expression "WITH" license-exception-id /

                compound-expression "AND" compound-expression /

                compound-expression "OR" compound-expression ) /
                
                "(" compound-expression ")" )

license-expression =  1*1(simple-expression / compound-expression / UNLICENSED)
```

Notable thing here, all of the syntax is **case-sensitive**. This is defined by SPDX.
Namely `MIT or Apache-2.0` is not a valid expression.

LicenseRef and DocumentRef will not be implemented in this iteration.
Custom licenses in combination with a standard licenses, are treated as custom licenses and as such they will need to be embedded in the package. 

#### Approach for in-house packages - UNLICENSED

We believe that every NuGet package in the world should have either a license embedded or a License Identifier specified.
When neither of the above two options is specified the NuGet client will warn. To improve this experience, we allow customers to specify "UNLICENSED" as a valid identifier.
In this approach we are borrowing some learnings from NPM. This means that the package does not grant any permissions for its re-use.
Public feeds would normally reject such packages.

### Implementation

A new model is defined, `NuGetLicenseExpression` which will be a parse tree of the above mentioned syntax.
`NuGetLicenseExpression::Parse(string licenseExpression)` will parse a NuGetLicenseExpression from a string.

If the expression has invalid syntax, the method will throw.
If the expression has deprecated identifiers, the method will throw.
If the expression has non standard license identifier (note that there's no concept of non-standard Exception identifiers), the parsing succeeds, and the object model (NuGetLicense), clearly represents that.
The implementation will also have an embedded list of valid license identifiers and exceptions.

### Nuspec schema changes

There are various options that were discussed regarding the nuspec schema and the pack scenarios.
A couple of ground rules:
The license file and license expression are exclusive.
We define a new element called `license`.
- Required type attribute, that has to be "type" or "expression" in this implementation. 
- Optional version attribute. This attribute defines the license expression grammar version. 
This will only be relevant if in the future we have an update to the license expression grammar.
- The value of the element is considered the license value. 
 
```
<xs:element name="license" maxOccurs="1" minOccurs="0">
    <xs:complexType>
    <xs:simpleContent>
        <xs:extension base="xs:string">
        <xs:attribute name="type" type="xs:string" use="required"/>
        <xs:attribute name="version" type="xs:string" use="optional"/>
        </xs:extension>
    </xs:simpleContent>
    </xs:complexType>
</xs:element>
```

#### License Expression pack

For .NET SDK based projects, we recommend that you use dotnet.exe to pack.
We will add a new property to the project file. 

`PackageLicenseExpression`

To specify a license expression, you can do it as below:

```
<PropertyGroup>
    <PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>
</PropertyGroup>
```

A sample nuspec would be the following:

```
<?xml version="1.0"?>
<package>
  <metadata>
    <id>LicensesExample</id>
    <version>1.0.0</version>
    <authors>NuGet</authors>
    <owners>NuGet</owners>
    <requireLicenseAcceptance>true</requireLicenseAcceptance>
    <description>A license example package.</description>
    <license type="expression">Apache-2.0</license>
  </metadata>
</package>
```

If you want to add a license expression with a different license expression grammar version, use the PackageLicenseExpressionVersion. This corresponds to the optional version attribute of the license element.
We strongly encourage that you don't define the license expression version.

```
<PropertyGroup>
    <PackageLicenseExpression>Apache-2.0</PackageLicenseExpression>
    <PackageLicenseExpressionVersion>1.0.0</PackageLicenseExpressionVersion>
</PropertyGroup>
```

#### License file pack

To pack a license file, we follow a similar pattern to content files.
It's on the user to make sure that license file gets packed into the package. Pack will error if the license cannot be found.
We **strongly** recommend that that location is the root of the package to avoid potential future conflicts.
We also recommend that the license name is LICENSE[.txt|.md], all caps similar to the repositories convention. 

An example with an SDK based project is as below:

```
<PropertyGroup>
    <PackageLicenseFile>LICENSE.txt</PackageLicenseFile>
</PropertyGroup>

<ItemGroup>
    <None Include="licenses\LICENSE.txt" Pack="true" PackagePath=""/>
</ItemGroup>
```

Given a nuspec, with a file section like below, a sample packing manifest would be the following:

```
<?xml version="1.0"?>
<package>
  <metadata>
    <id>LicensesExample</id>
    <version>1.0.0</version>
    <authors>NuGet</authors>
    <owners>NuGet</owners>
    <requireLicenseAcceptance>true</requireLicenseAcceptance>
    <description>A license example package.</description>
    <license type="file">LICENSE.txt</license>
  </metadata>
  <files>
    <file src="bin\Release\**\*.dll" target="lib\" />
    <file src="licenses\LICENSE.txt" target="" />
  </files>
</package>
```

As called out in the earlier spec, the license and licenseUrl metadata cannot be combined.

#### Standard license list updates

If new licenses are added, customers would have to wait for a future release to specify that licenses or ignore this problem skipping all package validations.

#### Validations

The following table refers to the pack validations.

| license expression | license file | licenseUrl | pack behavior |
| ------------------ | ----------- | ---------- | ------------- |
| empty | empty | value | warn |
| value | value | irrelevant | error |
| value | empty | value | error |
| empty | value | value | error |
| valid expression with standard identifiers | empty | empty | success |
| UNLICENSED | empty | empty | success |
| expression with custom license identifier |  empty | empty | warn |
| expression with invalid syntax | irrelevant | irrelevant | error |
| expression with deprecated licenses | irrelevant | irrelevant | error |
| empty | file reference present in nupkg | empty | success |
| empty | file reference not present in nupkg | empty | error |

### Visual Studio experience

In Visual Studio, NuGet will do its best effort to parse the expression. Unknown licenses are irrelevant in this context.

To provide an optimal experience to the package consumers we will be doing a protocol update to allow the client to get the LicenseExpression.
We will only be updating the V3 protocol. The V2 protocol is not considered.

In the case that the license expression is not parseable, the client will display a warning to the end-user.

|  | License expression | License file (embedded) | LicenseUrl |
| - | - | - | - |
| Browse | NuGet will read new LicenseExpression field from search and [pretty display](#pretty-display-of-license-expressions) it. | The servers will not provide a value for the LicenseExpression field, so NuGet falls back to the LicenseUrl field. As a convenience NuGet.org will host the licenses. [License Hosting](#nuget.org-license-hosting) | The behavior is unchanged. |
| Installed | Read the LicenseExpression from the nuspec and [pretty display](#pretty-display-of-license-expressions) it. | Allow the user to view the embedded license extracted on disk. | The behavior is unchanged. |

#### NuGet.org License Hosting

In order to improve experience for our customers, NuGet.org will host the embedded licenses from packages and put that link in the LicenseUrl. The implications of this are described below:

| | VS new clients | VS old clients |
|-|----------------|----------------|
|Browse|The client will display the link to the hosted license| The client will display the link to the hosted license|
|Installed| The client will allow the customer to view the license already on disk| The client will read the NuGet tools embedded link which will provide them with instructions on how to view their license.|

#### Pretty display of License Expressions

Given a complex expression such as `MIT or Apache-2.0`, to provide an experience on par with the earlier license implementation, the client will provide links to the content of MIT and Apache-2.0 themselves.

To do this, we will be introducing a new service to host the standard license text.
While managed by Microsoft and NuGet, this service is otherwise not connected to the NuGet Gallery.
Regardless from which feed local/NuGet.org/3rd party the links will lead to this service.

The link to that service will be licenses.nuget.org and a sample link to a license would be.

> https://licenses.nuget.org/Apache-2.0.html

#### NuGet Tools Embedded link

The purpose of this link to handle combinations of down-level clients and out of the date servers.

The below table describes how it contributes to the overall experience.

| | VS new clients | VS old clients |
|-|----------------|----------------|
|Browse|If the server is not hosting/advertising license expressions, the client shows the NuGet tools embedded link with instructions on how to find the license in question. | Whether the server sends license expressions is irrelevant. If the server is not hosting its own licenses/expressions  the client shows the NuGet tools embedded link with instructions on how to find the license in question. |
|Installed| N/A. The client will read the license expression/file and show that. | The clients shows the NuGet tools embedded with instructions on how to find the license in question.|

### References

* SPDX issue talking about the case sensitive of identifiers. [Link](https://github.com/spdx/spdx-spec/issues/63) that talks about case sensitivity.
* The full SPDX specification can be found [Link](https://spdx.org/sites/cpstandard/files/pages/files/spdxversion2.1.pdf)
* SPDX FAQ [Link](https://spdx.org/frequently-asked-questions-faq-0)
* The SPDX License expressions syntax. [Link](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60)
* The standard list of SPDX licenses. [Link](https://spdx.org/licenses/)
* The standard list of SPDX exceptions. [Link](https://spdx.org/licenses/exceptions-index.html)
* How to request a new license/exception to be added to the standardized lists. [Link](https://spdx.org/spdx-license-list/request-new-license)
* The SPDX license list can be found on GitHub as well [Link](https://github.com/spdx/license-list-data)
* Recommendations on accessing the licenses programatically [Link](https://spdx.org/sites/cpstandard/files/pages/files/spdx-tr-2014-2_v1_0_access_license_list.pdf)
* Details about the license handling in NPM. [Link](https://docs.npmjs.com/files/package.json#license)

#### Known limitations

* The client will be warning for unknown licenses and there will be some lag in the list update. However this not blocking pack, so it's of lower importance.