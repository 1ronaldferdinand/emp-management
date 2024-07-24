# emp-management - Vue 2 Project with JSON Server

### Features

- **Vue 2** for frontend development
- **JSON Server** for backend API simulation
- **Axios** for HTTP requests

## Installation

### Prerequisites

- Node.js and npm installed on your machine
- Node.js => 18.13.0^
- NPM => 8.19.3^

#### Install Dependencies
```
npm install
```

#### Run Json-Server 
```
json-server data/db.json
```
- This will start the JSON Server at https://localhost:3000.
- By default in local, JSON Server will run at host localhost and port 3000. 
- If your host or port was different you need to update .env file.

#### Run the Vue development server
```
npm run serve
```
- The Vue development server will start at http://localhost:8080.

## Project Structure
```
├── data
│   └── db.json
├── public
│   ├── favicon.ico
│   └── index.html
├── src
│   ├── assets
│   ├── components 
│   ├── App.vue
│   └── main.js
├── .env
├── package.json
└── README.md
```
