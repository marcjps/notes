# MySQL

#### Command line client

Connect to localhost:

	mysql -p

#### Change database

	use <database>;

#### List tables 

	show tables;

#### Show columns

	describe <tablename>;

#### Create a table with primary and foreign keys

	CREATE TABLE user_address (
		id INT NOT NULL AUTO_INCREMENT, 
		user_id INT, 
		address VARCHAR(255), 

		PRIMARY KEY (ID), 
		FOREIGN KEY (user_id) 
			REFERENCES user(id) 
	);

#### Create a view

	CREATE VIEW UserAddress AS SELECT u.name, u.email, ua.address from user u Join user_address ua on ua.user_id = u.id;         

