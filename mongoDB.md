# Create Users in Mongo
```
db.createUser({user:"userName", pwd:"Password",roles:[{role:"readWrite",db:"databaseName"}]})

db.createUser({user:"devkittens", pwd:"5585B1722E9206A4704061E26A2E69F560487768C150D5501039CE7F9693A179",roles:[{role:"readWrite",db:"devkittens"}]})
```
# Edit 
```
mongodb://{userName}:{password}@localhost:27017/{databaseName}  

mongodb://devkittens:5585B1722E9206A4704061E26A2E69F560487768C150D5501039CE7F9693A179@localhost:27017/devkittens
```
