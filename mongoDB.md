# Create Users in Mongo

We'll have to create a user in the server's mongoDB.  
Log into the server as an admin. Select the database that you want to use. Then create a user for that database.
```
use DATABASE_NAME
db.createUser({user:"USER_NAME", pwd:"PASSWORD",roles:[{role:"readWrite",db:"DATABASE_NAME"}]})

use devkittens
db.createUser({user:"devkittens", pwd:"5585B1722E9206A4704061E26A2E69F560487768C150D5501039CE7F9693A179",roles:[{role:"readWrite",db:"devkittens"}]})
```
# Edit your mongoDB connection URI
In your code, you'll want to edit you mongoDB connection URI.  If you don't want to create users on your local
machine, put the connection string either a configuration file or enviromental variable.
```
mongodb://{USER_NAME}:{PASSWORD}@localhost:27017/{DATABASE_NAME}  

mongodb://devkittens:5585B1722E9206A4704061E26A2E69F560487768C150D5501039CE7F9693A179@localhost:27017/devkittens
```
