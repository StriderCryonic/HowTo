# Implementing a Rest Server using Python Flask and MySQL

### Step 1: Setting up the base Rest server:
* Create a python file. Inside it, enter the following code:
	>
	> ```from flask import Flask``` <br>
	> ```app = Flask(__name__)``` <br>
	> ```@app.route('/')``` <br>
	> ```def hello_world():``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```return 'Hello, Docker!'``` <br>
	> ``` if __name__ == "__main__":``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp; ```app.run(host = '0.0.0.0', port = 8000)```
	>

* The above program allows us to visit ```localhost:8000```, or essentially the TCP port that the python flask app is running on, and recieve a response on the page as "Hello, Docker!"
* The output of the above can also be recieved via a console command, ```curl localhost:8000```

### Step 2: Creating the volumes and MySQL container
* To create the volumes (persistent data storage), we use the commands:
	> ```docker volume create mysql```<br>
	> ```docker volume create mysqlconfig```
* We also need to create a network to connect these volumes, the rest server as well as the MySQL database.
	> ```docker network create mysqlnet```
* We are then able to run the MySQL image, which is a default image available in docker. Using the run command is enough for the image to be created, there is no need for a build command.
	> ```docker run -d -v mysql:/var/lib/mysql -v mysql_config:/etc/mysql -p 3300:3300 --network mysqlnet --name mysqldb -e MYSQL_ROOT_PASSWORD=p@ssw0rd1 mysql```
	* The above function uses -v to connect the container to a volume, and --network to connect it to a bridge network.
	* The -e is used in order to properly declare the root password in order to access the database through the python program. This argument can be removed in order to use the inbuilt sql password (if exists).

### Step 3: Creating the /initdb page to initialize database in python
* To create the page that initializes the database:
	>
	> ```@app.route('/')``` <br>
	> ```def db_init():``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```mydb = mysql.connector.connect(host = chost, user = cuser, password = cpasswd)``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```cursor = mydb.cursor()``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```cursor.execute("CREATE DATABASE IF NOT EXISTS {};".format(cdatabase))``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```mydb = mysql.connector.connect(host = chost, user = cuser, password = cpasswd, database = cdatabase)``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```cursor = mydb.cursor()``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```cursor.execute("CREATE TABLE IF NOT EXISTS widgets(ID number(2) PRIMARY KEY, name VARCHAR(255) NOT NULL, description VARCHAR(255), json BLOB, IPv4 binary(4));")``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```mydb.commit()``` <br>
	> &nbsp;&nbsp;&nbsp;&nbsp;```return 'initialized database'```
	>

	
	* This creates the widgets table if it does not already exist with the datatypes of number for the ID, VARCHAR for the name and description, a JSON blob format, and a slot for IPv4 entry. \
		* Note: IPv4 is simply the display conversion of the binary content in the text given. i.e 192.65.68.201 => 0xC04144C9

	* In the above code, the variables chost, cuser, cpasswd, and cdatabase are taken from a config file made in the same directory. When this program becomes a container, this data is moved along with it, and will run without any issues.
		* Adding config file parsing is simply a matter of importing the configparser module in python, and running the ```configParser.RawConfigParser().read(configfile)``` function.```
		* Now the config file can be changed to ensure that the database and its details can be modified and are not hardcoded into the system.
### Step 4: Adding data.

* It is then possible to post data from a front end to programatically add data into the database in a similar fashion to the creation of the table in the code snippet above.


### Note:

* All the data that is stored into the database is stored in the volume that was created in the Step 2, meaning that all data is persistent even if the container is stopped and restarted.
* The program is able to parse all kinds of data as long as they are saved into a string format (in python) and moved as an attribute for a .format() function.
