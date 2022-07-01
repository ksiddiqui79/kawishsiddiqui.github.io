title: Encryption & Decryption UDF in Amazon Redshift 
description: Encryption & Decryption UDF in Amazon Redshift
keywords: encrypt, decryption, decrypt, encryption, udf, amazon, function, utility, vacuum, analyze, data lake, datalake, formation, case study, shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data

<script>
 </script>
 ---
!!! note "Important"  
     This post is to help customers who are looking for AES encryption and decryption UDF implementation in Amazon Redshift to answer "How AES encryption and decryption UDF can be implemented in Amazon Redshift?".
     This post should not be considered as "Best Practice" for such implementation.

  
# How to implement AES Encryption &amp; Decryption UDF in Amazon Redshift

# What is this post about?

This post is to help customers who are looking for AES encryption and decryption UDF implementation in Amazon Redshift to answer &quot;How AES encryption and decryption UDF can be implemented in Amazon Redshift?&quot;.

Beside Amazon Redshift cluster level encryption, from data governance and protection perspective often customers want to use [Advanced Encryption Standard (AES)](https://www.nist.gov/publications/advanced-encryption-standard-aes?pub_id=901427)for their sensitive data to be stored in Redshift database columns. As data loading is part of ELT, this is often helpful for ELT developers and architects to simplify their ELT process for Amazon Redshift and reducing development efforts using User Defined Function where developers need to encrypt data. At the same time, authorized users and data scientist can also leverage the decrypted data using decryption UDF.

# Advanced Encryption Standard (AES)

The [Advanced Encryption Standard (AES)](https://www.nist.gov/publications/advanced-encryption-standard-aes?pub_id=901427) is a specification for the encryption of electronic data established by the U.S. National Institute of Standards and Technology (NIST) in 2001.

# UDF Deployment

These user defined functions use [PyPI pyaes ](https://pypi.org/project/pyaes/)module to encrypt and decrypt data using AES encrypt and decrypt methods.

AES encryption supports 128 bits (16 bytes), 192 bits (24 bytes) or 256 bits (32 bytes), any key length other than supported will throw error during encryption and/or decryption.

See for more details: [https://github.com/ricmoo/pyaes](https://github.com/ricmoo/pyaes)

## Create library
 
```sql
CREATE OR REPLACE LIBRARY pyaes
LANGUAGE plpythonu
FROM 'https://tinyurl.com/redshift-udfs/pyaes.zip?raw=true';
;
```
 
## Create encrypt function
 
```sql
--- Create a separate schema to deploy encryption UDF
CREATE SCHEMA udf_enc;

--- Create UDF
CREATE OR REPLACE FUNCTION udf_enc.aes_encrypt(input VARCHAR(max), vKey VARCHAR(256))
RETURNS VARCHAR STABLE AS $$
  import pyaes
  import binascii
  if input is None:
     return None
  key = vKey # Your Key here
  aes=pyaes.AESModeOfOperationCTR(key)
  cipher_txt=aes.encrypt(input)
  cipher_txt2=binascii.hexlify(cipher_txt)
  return str(cipher_txt2.decode('utf-8'))
$$ LANGUAGE plpythonu ;
```
 
## Create decrypt function
 
```sql
--- Create a separate schema to deploy Decryption UDF to control decryption access to only authorized users.
CREATE SCHEMA udf_dec;

--- Create UDF
CREATE OR REPLACE FUNCTION 
udf_dec.aes_decrypt(encrypted_msg varchar(max), vKey VARCHAR(256))
RETURNS VARCHAR STABLE AS $$
  import pyaes
  import binascii
  if encrypted_msg is None or len(str(encrypted_msg)) == 0:
       return None
  key = vKey # Your decryption key here
  aes = pyaes.AESModeOfOperationCTR(key)
  encrypted_msg2=binascii.unhexlify(encrypted_msg)
  decrypted_msg2 = aes.decrypt(encrypted_msg2)
  return str(decrypted_msg2.decode('utf-8'))
$$ LANGUAGE plpythonu ;
```
 
## Test functionality

### Setup Test Environment and Data
 
```SQL
--- Create schema to contain sensitive data
CREATE SCHEMA secure_edw_t;

--- Create views schema
CREATE SCHEMA edw_v;

--- Create a sample user
CREATE USER edw_v_read password 'B!@nK73x7';

--- Grant USAGE privileges on schema to make object visible to user (this will NOT make data visible to user)
GRANT USAGE ON SCHEMA secure_edw_t TO edw_v_read;
GRANT USAGE ON SCHEMA edw_v TO edw_v_read;

--- Create table
CREATE TABLE secure_edw_t.emp_secure
(
   emp_id int,
   emp_name VARCHAR(255),
   emp_phone varchar(255),
   emp_name_enc VARCHAR(255),
   emp_phone_enc varchar(255)
);

--- Create view to project only encrypted values to the user
CREATE VIEW edw_v.emp
AS
(
SELECT emp_id
     , emp_name_enc as emp_name
     , emp_phone_enc as emp_phone
FROM secure_edw_t.emp_secure );

GRANT SELECT ON edw_v.emp TO edw_v_read;

--- Populate test data in Table
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (1, 'Azhar', '2341232345');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (2, 'Humayun',  '4323445676');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (3, 'Rawish', '2221233213');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (4, 'Khayyam', '9808987658');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (5, 'Kawish', '2342342456');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (6, 'Shariq', '6575768475');
INSERT INTO secure_edw_t.emp_secure (emp_id, emp_name, emp_phone)
Values (7, 'Qasim', '3213453234');
```
 
See [GRANT](https://docs.aws.amazon.com/redshift/latest/dg/r_GRANT.html) for more details about granting appropriate permission in Amazon Redshift.

### Encrypting Data

In this simple test, we are using different keys for corresponding columns for encryption.

- emp_name is encrypted using key ```empnameKey/fhci4=dnv73./xorb3f05```
- emp_phone is encrypted using key ```empphoneKey29s0vne03]jv023n=bn34```

**SQL**
 
```SQL
--- Encrypt data
UPDATE secure_edw_t.emp_secure
SET emp_name_enc = udf_enc.aes_encrypt(emp_name, LPAD('empnameKey/fhci4=dnv73./xorb3f05', 32, 'z')),
emp_phone_enc = udf_enc.aes_encrypt(emp_phone, LPAD('empphoneKey29s0vne03]jv023n=bn34', 32, 'z'))
;

SELECT * FROM secure_edw_t.emp_secure;
```

**Result**

Actual data in table will be like below:

| **emp\_id** | **emp\_name** | **emp\_phone** | **emp\_name\_enc** | **emp\_phone\_enc** |
| --- | --- | --- | --- | --- |
| 1 | Azhar | 2341232345 | e8a769c636 | 9345901d200893aeca19 |
| 2 | Humayun | 4323445676 | e1a86cc63d3194 | 9545961f260f94abc91a |
| 3 | Rawish | 2221233213 | fbbc76ce372c | 9344961d200892afcf1f |
| 4 | Khayyam | 9808987658 | e2b560de3d2597 | 984e94142b0396abcb14 |
| 5 | Kawish | 2342342456 | e2bc76ce372c | 9345901e210f93a9cb1a |
| 6 | Shariq | 6575768475 | fab560d52d35 | 97439319250d99a9c919 |
| 7 | Qasim | 3213453234 | f8bc72ce29 | 9244951f260e92afcd18 |

### Test decryption and user data visibility

#### Test values decryption

##### Using correct key

**SQL**
 
```SQL
SELECT emp_id,
       udf_dec.aes_decrypt(emp_name, LPAD('empnameKey/fhci4=dnv73./xorb3f05', 32, 'z')) emp_name,
       udf_dec.aes_decrypt(emp_phone, LPAD('empphoneKey29s0vne03]jv023n=bn34', 32, 'z')) emp_phone
FROM edw_v.emp;
```

**Result**

| **emp\_id** | **emp\_name** | **emp\_phone** |
| --- | --- | --- |
| 1 | Azhar | 2341232345 |
| 2 | Humayun | 4323445676 |
| 3 | Rawish | 2221233213 |
| 4 | Khayyam | 9808987658 |
| 5 | Kawish | 2342342456 |
| 6 | Shariq | 6575768475 |
| 7 | Qasim | 3213453234 |

#### Using incorrect key

**SQL**
 
```SQL
SELECT emp_id,
       udf_dec.aes_decrypt(emp_name, LPAD('empWRONGKey/fhci4=dnv73./xorb3', 32, 'z')) emp_name,
       aes_decrypt(emp_phone, LPAD('phoneWRONGKey29s0vne03]jv023n=', 32, 'z')) emp_phone
FROM edw_v.emp;
```

**Result**

```
SQL Error [500310] [XX000]: [Amazon](500310) 
Invalid operation: 
    UnicodeDecodeError: 'utf8' codec can't decode 
    byte 0xfb in position 0: invalid start byte. 
Please look at svl_udf_log for more information
Details:
-----------------------------------------------

  error:  UnicodeDecodeError: 'utf8' codec can't decode byte 0xfb in position 0: invalid start byte. Please look at svl_udf_log for more information
  code:      10000
  context:   UDF
  query:     3405510
  location:  udf_client.cpp:369
  process:   query0_2002_3405510 [pid=43012]

  -----------------------------------------------;
```

### **Check for error details if error occurred**

```SQL
SELECT * FROM svl_udf_log;
```

See [svl\_udf\_log](https://docs.aws.amazon.com/redshift/latest/dg/r_SVL_UDF_LOG.html) for more details

- Test data visibility for user

#### Change session user

We are using ```SET SESSION AUTHORIZATION``` command to switch user session temporary to "edw\_v\_read". If you are not running this test using superuser, then you have to logoff and login using the "edw_v_read" credentials to test.

See [SET SESSION AUTHORIZATION](https://docs.aws.amazon.com/redshift/latest/dg/r_SET_SESSION_AUTHORIZATION.html) for more details.

```SQL
SET SESSION AUTHORIZATION 'edw_v_read';
```

#### Read data from view

**SQL**

```SQL
SELECT *
FROM edw_v.emp;
```

**Result**

| **emp\_id** | **emp\_name** | **emp\_phone** |
| --- | --- | --- |
| 1 | e8a769c636 | 9345901d200893aeca19 |
| 2 | e1a86cc63d3194 | 9545961f260f94abc91a |
| 3 | fbbc76ce372c | 9344961d200892afcf1f |
| 4 | e2b560de3d2597 | 984e94142b0396abcb14 |
| 5 | e2bc76ce372c | 9345901e210f93a9cb1a |
| 6 | fab560d52d35 | 97439319250d99a9c919 |
| 7 | f8bc72ce29 | 9244951f260e92afcd18 |

#### Attempt to read data from table

**SQL**

```SQL
SELECT *
FROM secure_edw_t.emp_secure;
```

**Result**

```
SQL Error [500310] [42501]: [Amazon](500310) 
Invalid operation: permission denied for relation emp_secure;
```
