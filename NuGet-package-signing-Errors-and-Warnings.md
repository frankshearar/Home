This document contains a list of all warnings and errors that may occur during signing, verifying and using signed packages.

Package signing related errors and warnings should be in the following range -

| Log Message Type | Starting Code | Ending Code |
|------------------|---------------|-------------|
| All              | NU3000        | NU3999      |
| Errors           | NU3000        | NU3499      |
| Warnings         | NU3500        | NU3999      |

Further, error code ranges are sub divided as follows - 

| Error/Warning range | Sub-Group |
|---------------------|-----------|
| 3000/3500           |           |
| 3100/3600           |           |
| 3200/3700           |           |
| 3300/3800           |           |
| 3400/3900           | Timestamp |

# Errors

## Generating author signature

### NUxxxx

#### Issue
issue explanation goes here

## Generating timestamp

### NU3401

#### Issue
Author certificate was not valid when it was timestamped by the timestamp authority.


### NU3402

#### Issue
Timestamp authority's certificate does not build to a trusted root authority.


### NU3403

#### Issue
Timestamp authority's certificate does not have a valid Enhanced Key Usage field.


### NU3404

#### Issue
Timestamp authority's response does not contain the right author signature value hash.

### NU3405

#### Issue
Timestamp authority's response does not contain the right nonce that is used to track the request.

### NU3406

#### Issue
Timestamp authority's response does not contain the right hash algorithm identifier for the author signature.

## Verifying author signature

### NUxxxx

#### Issue
issue explanation goes here

<br/>
# Warnings 

## Generating author signature

### NUxxxx

#### Issue
issue explanation goes here

## Generating timestamp

### NU3901

#### Issue
Timestamp authority's response does not contain the right author signature value hash.

## Verifying author signature

### NUxxxx

#### Issue
issue explanation goes here