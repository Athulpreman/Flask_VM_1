# Flask_VM_1


Sign up and create a Ubuntu virtual machine (B1s)

https://azure.microsoft.com/en-gb/free/students/ 

Setup - run the following commands 

sudo apt update

sudo  apt -y upgrade

sudo apt -y install python3-pip

sudo apt -y install python3-flask

mkdir venv

cd venv

sudo apt -y install python3.12-venv

python3 -m venv .

#Create a repository in GitHub

#Add a file hello.py to the repo on GitHub

==================

hello.py

===================

from flask import Flask
app = Flask(__name__)
@app.route("/")#URL leading to method
def hello(): # Name of the method
 return("Hello World!") #indent this line
if __name__ == "__main__":
 app.run(host='0.0.0.0', port='8080') # indent this line

====================

#Click commit to main

#Click code 

#Copy the .git link

#git clone <repo link> # it will say cloning into <X>

cd <X>

run:

python3 hello.py
#ensure port 8080 is open on portal.azure.com 

then in Chrome, visit http://<IP>:8080

you should see 

Hello World!

#Returning to terminal, run:

#Let's get Flask running on https:
#run
sudo apt -y install apache2
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
#make sure you have configured you DNS (hostname) on Azure
sudo certbot --apache
sudo cp /etc/letsencrypt/live/<hostname>/cert.pem .
sudo cp /etc/letsencrypt/live/<hostname>/privkey.pem .
sudo chown `whoami` *.pem
#edit hello.py on GitHub 
change the last line to:

app.run(host='0.0.0.0',port='8080', ssl_context=('cert.pem', 'privkey.pem')) #Run the flask app at port 8080

run
python3 hello.py
==========================

#Let's get some parameters:
#Create file greeting.py on GitHub as follows:
from flask import Flask
from flask import request
app = Flask(__name__)
@app.route("/")#URL leading to method
def hello(): # Name of the method
 return("Hello World!")
@app.route("/greetme")#different URL
def helloall(): # different method name
 name = request.args.get('name')#retrieve GET parameters
 return("Hello {}!".format(name))#Python’s string.format
if __name__ == "__main__":
 app.run(host='0.0.0.0',port='8080', ssl_context=('cert.pem', 'privkey.pem')) #Run the flask app at port 8080

============

python3 greeting.py 

Test with https://yourdomain:8080/greetme?name=yourname

==========================================

set up DB unless already done

sudo apt -y install mariadb-server mariadb-client 

../bin/pip3 install flask_cors mysql-connector-python

sudo mysql

====


CREATE USER 'web'@'localhost' IDENTIFIED BY 'webPass';
GRANT ALL PRIVILEGES ON *.* to 'web'@'localhost';

CREATE DATABASE student; 
USE student;
CREATE TABLE students (studentName VARCHAR(255), email VARCHAR(255), studentID INT NOT NULL AUTO_INCREMENT,
PRIMARY KEY(studentID));

INSERT INTO students (studentName, email) values ("first student", "firststudent@mydbs.ie"); 
INSERT INTO students (studentName, email) values ("second student", "secondstudent@mydbs.ie ");
SELECT * FROM students;
exit;

==========================================

Now to set up the API

==========================================

#On GitHub, create templates/add.html


<!DOCTYPE html>

<html>

<head>

<meta charset="utf-8">

<meta name="viewport" content="width=device-width">

<title>Add Item</title>

</head>

<body>

  <form action="/add" method="POST">

    Name: <input type="text", name="name">

    Email: <input type="text", name="email">

    <button type="submit" name="submit">Add</button>

  </form>    

</body>

</html>



#On GitHub, create app.py

#paste:

from flask import Flask
from flask import render_template
from flask import request
import mysql.connector
from flask_cors import CORS
import json
mysql = mysql.connector.connect(user='web', password='webPass',
  host='127.0.0.1',
  database='student')

from logging.config import dictConfig

dictConfig({
    'version': 1,
    'formatters': {'default': {
        'format': '[%(asctime)s] %(levelname)s in %(module)s: %(message)s',
    }},
    'handlers': {'wsgi': {
        'class': 'logging.StreamHandler',
        'stream': 'ext://flask.logging.wsgi_errors_stream',
        'formatter': 'default'
    }},
    'root': {
        'level': 'INFO',
        'handlers': ['wsgi']
    }
})
app = Flask(__name__)
CORS(app)
# My SQL Instance configurations
# Change the HOST IP and Password to match your instance configurations

@app.route("/add", methods=['GET', 'POST']) #Add Student
def add():
  if request.method == 'POST':
    name = request.form['name']
    email = request.form['email']
    print(name,email)
    cur = mysql.cursor() #create a connection to the SQL instance
    s='''INSERT INTO students(studentName, email) VALUES('{}','{}');'''.format(name,email)
    app.logger.info(s)
    cur.execute(s)
    mysql.commit()
  else:
    return render_template('add.html')

  return '{"Result":"Success"}'
@app.route("/") #Default - Show Data
def hello(): # Name of the method
  cur = mysql.cursor() #create a connection to the SQL instance
  cur.execute('''SELECT * FROM students''') # execute an SQL statment
  rv = cur.fetchall() #Retreive all rows returend by the SQL statment
  Results=[]
  for row in rv: #Format the Output Results and add to return string
    Result={}
    Result['Name']=row[0].replace('\n',' ')
    Result['Email']=row[1]
    Result['ID']=row[2]
    Results.append(Result)
  response={'Results':Results, 'count':len(Results)}
  ret=app.response_class(
    response=json.dumps(response),
    status=200,
    mimetype='application/json'
  )
  return ret #Return the data in a string format
if __name__ == "__main__":
  #app.run(host='0.0.0.0',port='8080') #Run the flask app at port 8080
  app.run(host='0.0.0.0',port='8080', ssl_context=('cert.pem', 'privkey.pem')) #Run the flask app at port 8080

====================================================



../bin/python3 app.py # from inside the repository

Let's try a JS client in the browser

sudo nano /var/www/html/index.html

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width">
<title>Test API Client</title>
<script>
let doIt=()=>{
  let tab=document.getElementById("tab1");
  let rows=tab1.getElementsByTagName('tr');
  fetch('https://<yourdomain>:8080/')
    .then(response => response.json())
    .then(data=>data.Results.forEach(  //.slice(0,3)
      x=>{
        let newRow=rows[0].cloneNode(true);
        let divs=newRow.getElementsByTagName('td');
        divs[0].innerHTML=x['ID'];
        divs[1].innerHTML=x['Name'];
        divs[2].innerHTML=x['Email'];
        tab1.appendChild(newRow);
      }
    )
  );
}
</script>
</head>
<body>
<button onClick="doIt()">Press me</button>
This is where the results turn up: <br/>
<table id='tab1' bgcolor='blue'>
<tr><td>ID</td><td>Name</td><td>Email</td></tr>
</table></body>
</html>

now visit your domain