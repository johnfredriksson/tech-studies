# **Dummy data for dummies - Users**
*By John Fredriksson, 2022-11-30*

---


You just built another project that integrates a SQL database that communicates with the front-end through an API, but theres no users in sight. How can we test our application with no data to fetch?

In this article we'll take a deeper look at ways to create larger amounts of data to fill our database. In all projects we develop, most likely, every data structure will be different from the last. While the code in this article might not be tailored to your exact needs, it serves as a guide to tackle similar situations.

The project we are building today are based on a bike rental service, your database has already been modeled but the front-end developers are tired of building the client side application without any data to display.

## **Create .SQL files with Python**

For our purpose Python is a great choice of programming language with its simplistic style, great number of modules and it is really simple to get started in.

When it comes to the output files to be read by the database, there's a few different ways to proceed.

### **CSV**

```
"FirstName","LastName","PhoneNumber","EmailAdress","Balance","Password"
"Gary","Nelson","+1-798-929-7028x0441","G.Nelson71@hotmail.com","388","password"
"Joseph","Hernandez","001-062-346-5114x34670","J.Hernandez92@hotmail.com","274", "password"
"...","...","...","...","...","..."
```

A CSV file is as the name implies, a **C**omma **S**eparated **V**alue file that we fill with our data and tell MYSQL to insert all rows into a specific table. While it might be a good option in some cases, it can raise a lot of [errors](https://stackoverflow.com/questions/7638090/load-data-local-infile-forbidden-in-php). Personally, I prefer to create formated DML-files.

### **DML.sql**

```sql
USE mydb
INSERT  INTO Users
	( FirstName, LastName, PhoneNumber, EmailAdress, Balance, Password )
VALUES
	('Gary', 'Nelson', '+1-798-929-7028x0441', 'G.Nelson71@hotmail.com', 388, 'password'),
	('Joseph', 'Hernandez', '001-062-346-5114x34670', 'J.Hernandez92@hotmail.com', 274, 'password'),
	( ... )
```
DML, or **D**ata **M**anipulation **L**anguage,  is a family of computer languages including commands permitting users to manipulate data in a database. This manipulation involves inserting data into database tables, retrieving existing data, deleting data from existing tables and modifying existing data.

Sounds like the perfect use-case for our mission today. As I built my database inside a docker container it's very easy to incorporate our DML-files into the setup when starting the container.

---

## **Generating Users with Faker**

The [Faker](https://faker.readthedocs.io/en/master/index.html) module is a Python package that generates fake data for you. Whether you need to bootstrap your database, create good-looking XML documents, fill-in your persistence to stress test it, or anonymize data taken from a production service, Faker is for you.

In this case we'll take a closer look at the [standard providers documentation](https://faker.readthedocs.io/en/master/providers.html) which has all the functionality that we need.

### **Install Faker**

```bash
python3 -m pip install faker
```

### **Create file structure**
```bash
mkdir mocking
mkdir sql
touch mocking/create_users.py
```
The folder 'mocking' contain our python scripts to generate .SQL files. The 'sql'-folder holds our DML-files. To generate our users, our code will reside in 'create_users.py'.

### **Get started**

Import the Faker module and initialize it.

```python
from  faker  import  Faker

fake = Faker()
```
### **Identify fields in table**

First of all we need to check our Users schema to find out which values we need.
```sql
CREATE  TABLE  IF  NOT  EXISTS  `mydb`.`Users` (
	`id`  INT  NOT  NULL AUTO_INCREMENT,
	`FirstName`  VARCHAR(45) NOT  NULL,
	`LastName`  VARCHAR(45) NOT  NULL,
	`PhoneNumber`  VARCHAR(45) NOT  NULL,
	`EmailAdress`  VARCHAR(45) NOT  NULL,
	`Balance`  INT  NOT  NULL,
	`Password`  VARCHAR(45) NOT  NULL,
	`PartialPayment`  TINYINT,
	PRIMARY  KEY (`id`),
	UNIQUE  INDEX  `emailAdress_UNIQUE` (`EmailAdress`  ASC) VISIBLE,
	UNIQUE  INDEX  `PhoneNumber_UNIQUE` (`PhoneNumber`  ASC) VISIBLE)
```

The '**id**' will be created automatically, so we don't need to worry about that. **PartialPayment** is allowed to be null, so we'll skip that one as well.

The remaining fields and their types are the following:
- FirstName: string
- LastName: string
- PhoneNumber: string
- EmailAdress: string
- Balance: integer
- Password: string

Next step is to create a function for each field that returns a random, but believable value with correct type.
<br>
<br>

### **FirstName**

The Faker has a provider called [Person](https://faker.readthedocs.io/en/master/providers/faker.providers.person.html), which generate person related data. To use it, we simply define the following function in our file.

```python
def  get_first_name():
	return  fake.first_name()
```
Ehh.. is that all? Yes, that's all.
<br>
<br>

### **LastName**

By now you might have already guessed it, we implement the following function

```python
def  get_last_name():
	return  fake.last_name()
```
<br>
<br>

### **PhoneNumber**

But what about a phone number? Since phone numbers have a lot off different structures depending on where in the world it originates from, you might have to tweak Faker a bit, you can read more in the [Phone number provider](https://faker.readthedocs.io/en/master/providers/faker.providers.phone_number.html).

In my case, I'll settle for a believable set of digits and characters, since the numbers has no effect on the application itself and the data field is just specified as a VARCHAR.
```python
def  get_phone_number():
	return  fake.phone_number()
```
<br>
<br>

### **EmailAdress**

In this function, we get to be more creative in our methods. I want an Email that uses their full name, a random year of birth, a random separator and a believable domain.

Import random.
```python
import random
```

I supply the function with the persons first and last name. I then create a list of possible separators. The variable 'address' is a string constructed of the first letter of their first name, a random separator from the list followed by their last name and a random integer between 50 and 99 to act as a birth year. The string ends with a '@' as we return the string concatenated with 'fake.free_email_domain()' which gives us a real domain name.
```python
def  get_email_adress(first_name, last_name):
	separators = [".","-","_"]
	address = first_name[0] + separators[random.randint(0, len(separators) - 1)] + last_name + str(random.randint(50,99)) + "@"

return  address + fake.free_email_domain()
```
<br>
<br>

### **Balance**

My users need a balance in order to pay for their bike rides, in this case i just set it to a realistic sum of what a rider is expected to have in their app wallet.

```python
def  get_balance():
	return  random.randint(0,1000)
```
<br>
<br>

### **Password**

As a password you are free to select any word or completely randomised set of characters as you want. I'm setting everyones password to 'password' in order to access all accounts in the database during development. Further down the project when the team implemented hashed passwords, I'll simply edit this line to equal the hashed version of 'password'.

```python
def  get_password():
	return  "password"
```
<br>
<br>

### **Put a user together**

Now i combine all functions above into a function that creates a 'user'.

I start by generating the full name, and then return a list with every value that makes up a user. In order to make the correct email, we need the full name to be created before the return as we re-use the name in the email-function.
```python
def  generate_user():
	first_name = get_first_name()
	last_name = get_last_name()
	return [first_name, last_name,
		get_phone_number(),
		get_email_adress(first_name, last_name),
		get_balance(),
		get_password()]
```
<br>
<br>

### **Put a user together, *many many times*, in a formated SQL-file**


Let's finish up our script by actually creating multiple users and output it to a file.

At the end of our script we start with these rows. '**open**' opens a file, if it doesn't exist, it will be created. In this case it tries to open the file we specify in the first argument. The second argument tells python what we want to do with the file, a '**w**' makes python know that we want to write to the file. Just be aware that opening a file with 'w' instantly deletes all previous data in that file.

With '**write**' we can write lines to the file. Here we write the lines needed to initiate the insert process.

The following code
```python
with  open('../sql/insert-1-users.sql', 'w') as  fh:
	fh.write("USE 'mydb'\n")
	fh.write("INSERT INTO Users\n")
	fh.write(" ( FirstName, LastName, PhoneNumber, EmailAdress, Balance, Password )\n")
	fh.write("VALUES\n")
```
Results in the following text
```
USE mydb
INSERT  INTO Users
	( FirstName, LastName, PhoneNumber, EmailAdress, Balance, Password )
VALUES
```
Looks like a great start. Now we need to create the users and format their data within parenthesis. Our goal is a lot of lines looking like

```
	('Firstname', 'Lastname', 'PhoneNumber', 'EmailAdress', Balance, 'Password'),
```
The only important note now is that the last row should have a ";" at the end instead of a "," to follow correct SQL syntax and avoid errors.

A counter to keep track of which iteration we are currently on, to know if its the last iteration and we need to swap ',' into a ';'.
```python
counter = 1
```
A loop where we specify how many users we want. In each iteration we generate a user, format its values into the kind of line we described earlier and add a +1 to our counter.
```python
for n in  range(1,1000):
	user = generate_user()
	symbol =  ","
	fh.write(f" ('{user[0]}', '{user[1]}', '{user[2]}', '{user[3]}', {user[4]}, '{user[5]}'){symbol}\n")
	counter +=  1
```
And a small check to determine if it's the last iteration.
```python
if counter ==  999:
	symbol =  ";"
```
Now we have the following structure.

```python
with  open('../sql/insert-1-users.sql', 'w') as  fh:
	fh.write("USE 'mydb'\n")
	fh.write("INSERT INTO Users\n")
	fh.write(" ( FirstName, LastName, PhoneNumber, EmailAdress, Balance, Password )\n")
	fh.write("VALUES\n")
	counter = 1
	for  n  in  range(1,1000):
		user = generate_user()
		symbol = ","
		if  counter == 999:
			symbol = ";"
		fh.write(f" ('{user[0]}', '{user[1]}', '{user[2]}', '{user[3]}', {user[4]}, '{user[5]}'){symbol}\n")
		counter += 1
```

We run our python script and see that a file called 'insert-1-users.sql' has appeared in th 'sql'-folder. If we open the file we can see our insert script with a 1000 generated users.
```sql
USE mydb
INSERT  INTO Users
	( FirstName, LastName, PhoneNumber, EmailAdress, Balance, Password )
VALUES
	('Gary', 'Nelson', '+1-798-929-7028x0441', 'G.Nelson71@hotmail.com', 388, 'password'),
	('Joseph', 'Hernandez', '001-062-346-5114x34670', 'J.Hernandez92@hotmail.com', 274, 'password'),
	('Christopher', 'Williams', '039-432-2691', 'C.Williams96@hotmail.com', 798, 'password'),
	('Anthony', 'Ward', '+1-024-924-7949x108', 'A.Ward89@yahoo.com', 217, 'password'),
	('Sabrina', 'Henderson', '(136)509-7164x7744', 'S-Henderson54@hotmail.com', 393, 'password'),
	('James', 'Smith', '(789)952-0266', 'J_Smith61@gmail.com', 262, 'password'),
	('William', 'Williams', '913.582.0076', 'W-Williams95@hotmail.com', 370, 'password'),
	('Connie', 'Mitchell', '175.805.9565', 'C-Mitchell93@hotmail.com', 233, 'password'),
	('Sheryl', 'Brooks', '+1-406-585-7079x333', 'S_Brooks74@gmail.com', 649, 'password'),
	('Mark', 'Bentley', '(908)322-4845', 'M_Bentley76@hotmail.com', 7, 'password'),
	('...', '...', '...', '...', ..., '...'),
	('Sarah', 'Rosales', '001-441-511-4264x395', 'S-Rosales60@yahoo.com', 272, 'password');
```
That's pretty cool.

The reason of the filename is the fact that I'm deploying my database in a docker container where i specified a folder to copy all sql scripts from and run them in alphabetic order, **insert-1-users** is translated to **(type of DML)-(order)-(table)**.
<br>
<br>

### **Summary**
We took a look at Python module Faker, implemented it in our solution. Identified all fields that makes a user, proceeded to define functions for each field. Defined a function that creates data for all fields by incorporating the previously defined functions. Opened a document with write option to manipulate the documents content, with the help of a loop create how many users we want. Saved the result output to said file.

As stated in the introduction, every project will most likely have a slightly different schema for its users. This purpose of this article is the get a general idea of how to tackle a task of creating thousands of fake users.
