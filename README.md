## MERN STACK TESTER APPLICATION ##

- Developed the backend of the application
- Had to set up MongoDB and register first
- Once registration was completed I followed the steps below to initiate teh backend of the Web Application.
    {Will need to update this file later}

Terminal Commands
1. Mkdir "File" (Name the file whatever you choose) 
2. Cd into "File" 
3. npm init -y 
4. npm install express cors mongodb dotenv
5. npm install -g nodemon 
6. Code . (Open File in VS Code)

Body
1. Change Package.Json  "type:" "module".
3. Create a new file in backend folder called "server.js"
	1. Import express & Cors
	2. Import api file (Resturants for this example) 
	3. call app variable making it equal to express()
	4. call App.use (several of them)

```
app.use(cors())
app.use(express.json())
app.use("/api/v1/restaurants", restaurants)
app.use("*", (req, res) => res.status(404).json({error: "not found}"}) )
```
	5. Export default app

1. Create .env File
	1. Pull UrI File from mongoDB
	2. RESTREVIEWS_DB_URI=mongodb+srv://hassan:8002@cluster0.bi74v.mongodb.net/sample_restaurants?retryWrites=true&w=majority
	3. RESTREVIEWS_NS=sample_restaurants
	4. PORT=5000

2. Create New File index.js
	1. import  app 
	2. Import Mongodb
	3. Import dotenv 
	4. Config dotenv
	5. access to mongo client from mongoDB
	6. Add Your Port number
	7. Access client
	8. Add catch with error
	9. add .then with app.listen for port 
	10. 
```
import app from "./server.js"
import mongodb from "mongodb"
import dotenv from "dotenv"

dotenv.config()
const MongoClient = mongodb.MongoClient

const port = process.env.PORT || 8000

MongoClient.connect(
    process.env.RESTREVIEWS_DB_URI,
    {
        maxPoolSize: 50,
        wtimeoutMS: 2500,
        useNewUrlParser: true
    }
)
.catch(err => {
    console.error('Error is happening 🤧', err.stack)
    process.exit(1)
})
.then(async client => {
    app.listen(port, () => {
        console.log(`listening on port ${port}`);
    })
})
```
3. Create a New folder called API
	1. Add a file called restaurants.route.js
	2. Import express from express
	3. create router equal express  dot router
	4. add routes as needed but add route stating ("hello World")
	5. export default router
```
import express from "express"
const router = express.Router()
router.route("/").get((req, res) => res.send("hello world"))
export default router
```
4. Test Server is working
		1. In terminal run: nodemon server.
		2. Go to local host 5000 to test in browser. 
		3. You sahould see Hello World on Dom using api/v1/reasturants URL
---
1. Create New Folder DAO
	1. Create file restaurantsDAO.js
		1. Add syntax
```
let restaurants
export default class RestaurantsDAO {
    static async injectDB(conn) {
        if (restaurants){
            return
        }
        try {
            restaurants = await conn.db(process.env.RESTREVIEWS_NS).collection("restaurants")
        } catch (e){
            console.error(
                `Unable to establish a collection handle in restaurantsDAO: ${e}`,
          )
            
        }
    }
    static async getRestaurants({
        filters = null, 
        page = 0, 
        restaurantsPerPage = 20, 
    } = {}) {
        let query 
        if (filters) {
            if ("name" in filters){
                query = { $text: {$search: filters["name"] } }
            } else if ("cuisine" in filters){
                query = {"cuisine": {$eq: filters["cuisine"]}}
            } else if ("zipcode" in filters){
                query = {"address.zipcode": {$eq: filters["zipcode"]}}
            }
        }
        
    }
}
```
		2. Call your variable Restaurant
		3. export the filename. 
		4. run static async  and injectDB
		5. replicate syntax as added and manipulate to fit the needed library
		6. Then you set up query by calling query 
		7. you follow several steps by running an if statement to name the types of query you want to call from MangoDb
	2.       Add pages and finish skip ages for DB
```
let restaurants
export default class RestaurantsDAO {
    static async injectDB(conn) {
        if (restaurants){
            return
        }
        try {
            restaurants = await conn.db(process.env.RESTREVIEWS_NS).collection("restaurants")
        } catch (e){
            console.error(
                `Unable to establish a collection handle in restaurantsDAO: ${e}`,
          )
            
        }
    }
    static async getRestaurants({
        filters = null, 
        page = 0, 
        restaurantsPerPage = 20, 
    } = {}) {
        let query 
        if (filters) {
            if ("name" in filters){
                //you have to set up the text feild within Mango DB to setup name and text field
                query = { $text: {$search: filters["name"] } }
            } else if ("cuisine" in filters){
                query = {"cuisine": {$eq: filters["cuisine"]}}
            } else if ("zipcode" in filters){
                query = {"address.zipcode": {$eq: filters["zipcode"]}}
            }
        }
        let cursor 
        try {
            cursor = await restaurants
            .find(query)
        } catch (e) {
            console.error(`Unable to issue find command, ${e}`)
            return {restaurantsList: [], totalNumRestaurants: 0};
        }
        const displayCursor = cursor.limit(restaurantsPerPage).skip(restaurantsPerPage * page)
        try{
            const restaurantsList = await displayCursor.toArrray()
            const totalNumRestaurants =  await restaurants.countDocuments(query)
            return { restaurantsList, totalNumRestaurants }
         }  catch (e){
            console.error(
                `Unable to convert cursor to array or problem counting documents, ${e}`
            )
            return {restaurantsList: [], totalNumRestaurants: 0 }
            
        }
    }
}
```
3. Save and make a push to GitHub

4.  in Resturant.route import resturant.controller
	1. in route add the resturant.Crtl


4. Set up Resturant.controller.js
```
import RestaurantsDAO from "../dao/restaurantsDAO.js"
export default class RestaurantsController {
  static async apiGetRestaurants(req, res, next) {
    const restaurantsPerPage = req.query.restaurantsPerPage ? parseInt(req.query.restaurantsPerPage, 10) : 20
    const page = req.query.page ? parseInt(req.query.page, 10) : 0
    let filters = {}
    if (req.query.cuisine) {
      filters.cuisine = req.query.cuisine
    } else if (req.query.zipcode) {
      filters.zipcode = req.query.zipcode
    } else if (req.query.name) {
      filters.name = req.query.name
    }
    const { restaurantsList, totalNumRestaurants } = await RestaurantsDAO.getRestaurants({
      filters,
      page,
      restaurantsPerPage,
    })
    let response = {
      restaurants: restaurantsList,
      page: page,
      filters: filters,
      entries_per_page: restaurantsPerPage,
      total_results: totalNumRestaurants,
    }
    res.json(response)
  }
  static async apiGetRestaurantById(req, res, next) {
    try {
      let id = req.params.id || {}
      let restaurant = await RestaurantsDAO.getRestaurantByID(id)
      if (!restaurant) {
        res.status(404).json({ error: "Not found" })
        return
      }
      res.json(restaurant)
    } catch (e) {
      console.log(`api, ${e}`)
      res.status(500).json({ error: e })
    }
  }
  static async apiGetRestaurantCuisines(req, res, next) {
    try {
      let cuisines = await RestaurantsDAO.getCuisines()
      res.json(cuisines)
    } catch (e) {
      console.log(`api, ${e}`)
      res.status(500).json({ error: e })
    }
  }
}
```