---
title: 'Encryption with symmetric keys & certificates'
date: 2019-04-18T16:00:51+01:00
author: Rich
layout: post
permalink: /encryption-with-symmetric-keys-certificates
categories:
  - sqlserver
tags:
  - dba
  - encryption
  - SQL
  - sqlserver
  - t-sql
---

Recently I was asked to look into the possibility of encrypting some of the column level data that is held within a SQL Server instance, not knowing where to start and not running SQL Server 2017 I set about looking for options, one of which was to use Symmetric Keys & Certificates to encrypt the data and in this post I am going to demonstrate how I did that and what I learned.

First up I created a table that would potentially hold sensitive information I could use for demonstration purposes.

<pre>     
	<code class="sql">
		USE EncryptionTest
		CREATE TABLE Patients
		(
		PatientID INT IDENTITY(1,1) NOT NULL,
		PatientNumber varchar(100),
		PatientNumber_Encrypted VARBINARY(MAX) NULL,
		NHSNumber varchar(12),
		NHSNumber_Encrypted VARBINARY(MAX) NULL,
		Forename varchar(50),
		Forename_Encrypted VARBINARY(MAX) NULL,
		Surname varchar(50),
		Surname_Encrypted VARBINARY(MAX)
		);
	</code>
</pre>

In this example, I am going to make use of Patients. _(None of the data used in this example is real)_

<pre>     
	<code class="sql">
		USE EncryptionTest
		GO
		CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongPassword1234!!';
		GO
	</code>
</pre>

I created a master key within the database, using a strong password, recording this somewhere safe is mandatory else the data will be lost.

<pre>     
	<code class="sql">
		USE EncryptionTest
		CREATE CERTIFICATE Cert1 WITH SUBJECT = 'Protect it all';
		GO
	</code>
</pre>

I needed a self-signed certificate to secure all the data with.

<pre>     
	<code class="sql">
		USE EncryptionTest
		GO
		CREATE SYMMETRIC KEY Key1 WITH ALGORITHM = AES_128 ENCRYPTION BY CERTIFICATE Cert1;
		GO
	</code>
</pre>

Once I had created the master key and certificate I could go ahead and build up the [Symmetric Key](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-symmetric-key-transact-sql?view=sql-server-2017)

### Inserting Some Data

<pre>     
	<code class="sql">
		USE EncryptionTest
		INSERT INTO Patients (PatientNumber,NHSNumber,Forename,Surname)
		VALUES
		('UR12345679','123456789','Bonza','Owl'),
		('UR22345679','223456789','Bonza','Hughes'),
		('UR32345679','323456789','Bonza','Davies'),
		('UR42345679','423456789','Bonza','Evans'),
		('UR52345679','523456789','Bonza','Parry'),
		('UR62345679','623456789','Bonza','Brown'),
		('UR72345679','723456789','Bonza','Williams'),
		('UR82345679','823456789','Bonza','Jones'),
		('UR92345679','923456789','Bonza','Bloggs');
	</code>
</pre>

Once I had the master key, certificate, and Symmetric Key in place I could go ahead and insert some dummy data into our patient's table.

![Initial Encryption Insert](/img/Encryption-Initial-Insert.png)

### Convert

<pre>     
	<code class="sql">
		OPEN SYMMETRIC KEY Key1
		DECRYPTION BY CERTIFICATE Cert1
		GO
		UPDATE Patients
		SET 
			PatientNumber_Encrypted = EncryptByKey (KEY_GUID('Key1'),PatientNumber),
			NHSNumber_Encrypted = EncryptByKey (KEY_GUID('Key1'),NHSNumber),
			Forename_Encrypted = EncryptByKey (KEY_GUID('Key1'),Forename),
			Surname_Encrypted = EncryptByKey (KEY_GUID('Key1'),Surname)
		FROM 
			Patients
		GO
		CLOSE SYMMETRIC KEY Key1;
		GO
	</code>
</pre>

This is where the magic happens.

First I need to ask SQL Server to initialize the Symmetric Key, which is decrypted using the certificate I created.

Now I need to update the sensitive columns by encrypting the data and copying that encrypted data into the column marked &#8220;\_encrypted&#8221;

Once completed I need to tell SQL Server to close up the Symmetric Key.

As you can see, the data has been encrypted

![](/img/Encryption-Initial-Insert-Encrypted.png)

### Drop & Roll

<pre>     
	<code class="sql">
		USE EncryptionTest
		GO
		ALTER TABLE Patients
		DROP COLUMN PatientNumber;
		GO
		ALTER TABLE Patients
		DROP COLUMN NHSNumber;
		GO
		ALTER TABLE Patients
		DROP COLUMN Forename;
		GO
		ALTER TABLE Patients
		DROP COLUMN Surname;
	</code>
</pre>

Now that the sensitive data has been encrypted, I can drop the plain text columns

![](/img/Encrption-Dropped-Columns.png)

### Let's Have A Read

<pre>     
	<code class="sql">
		OPEN SYMMETRIC KEY Key1
		DECRYPTION BY CERTIFICATE Cert1
		SELECT 
			CONVERT(varchar,decryptbykey(PatientNumber_Encrypted)) as 'PatientNumber',
			CONVERT(varchar,decryptbykey(NHSNumber_Encrypted)) as 'NHSNumber',
			CONVERT(varchar,decryptbykey(Forename_Encrypted)) as 'Forename',
			CONVERT(varchar,decryptbykey(Surname_Encrypted)) as 'Surname'
		FROM 
			Patients
		GO
		CLOSE SYMMETRIC KEY Key1
		GO
	</code>
</pre>

The above example shows how to read the data from the encrypted columns.

As you can see, the data is returned as expected.

![](/img/Encryption-Read-Back.png)

### Insert Some More

<pre>     
	<code class="sql">
		OPEN SYMMETRIC KEY Key1
		DECRYPTION BY CERTIFICATE Cert1
		INSERT INTO Patients (PatientNumber_Encrypted,NHSNumber_Encrypted,Forename_Encrypted,Surname_Encrypted)
		VALUES
		(
		ENCRYPTBYKEY(Key_Guid('Key1'),CONVERT(varchar,'UR102345679')),ENCRYPTBYKEY(Key_Guid('Key1'),CONVERT(varchar,'103456789')),ENCRYPTBYKEY(Key_Guid('Key1'),CONVERT(varchar,'Bonza')),ENCRYPTBYKEY(Key_Guid('Key1'),CONVERT(varchar,'Doe')))
		GO
	</code>
</pre>

The above example shows how to go about adding more data into the encrypted columns.

![](/img/Encryption-Add-More-Data.png)

<pre>     
	<code class="sql">
		OPEN SYMMETRIC KEY Key1
		DECRYPTION BY CERTIFICATE Cert1
		SELECT 
			CONVERT(varchar,decryptbykey(PatientNumber_Encrypted)) as 'PatientNumber',
			CONVERT(varchar,decryptbykey(NHSNumber_Encrypted)) as 'NHSNumber',
			CONVERT(varchar,decryptbykey(Forename_Encrypted)) as 'Forename',
			CONVERT(varchar,decryptbykey(Surname_Encrypted)) as 'Surname'
		FROM 
			Patients
		GO
		CLOSE SYMMETRIC KEY Key1
		GO
	</code>
</pre>

Finally, using the above you can see that PatientNumber UR102345679 was added successfully, encrypted and read back by SQL Server.

![](/img/Encryption-More-Data-Read-Back.png)

Performance wise this would be extremely horrible, it isn't recommended to encrypt a primary key column, in production, there would be another column that would be the primary key and live within the index however this is for demonstration purposes only.

### Backup Them Keys

It is a really good idea to back up the certificate, you can do this easily with <a href="http://dbatools.io" target="_blank" rel="noopener noreferrer">DbaTools</a>

<pre>     
	<code class="sql">
		Backup-DbaDBCertificate -SqlInstance localhost -Database EncryptionTest -Path C:\Temp\Certs -EncryptionPassword (ConvertTo-SecureString -Force -AsPlainText StrongPassword123!)</</</code>
</pre>

In addition, you may want to back up the master key too

<pre>     
	<code class="powershell">
		Backup-DbaDBMasterKey -SqlInstance localhost -Database EncryptionTest -Path C:\Temp\Certs
	</code>
</pre>

This will prompt for a password, once provided, the master key will be backed up.
