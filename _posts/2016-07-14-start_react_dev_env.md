---
layout: post
title: Start react dev environment
---

origin video from: [https://egghead.io/lessons/react-react-fundamentals-development-environment-setup](https://egghead.io/lessons/react-react-fundamentals-development-environment-setup)

	npm install react react-dom --save
	npm install babel-loader babel-core babel-preset-es2015 babel-preset-react --save-dev
	touch index.html App.js main.js webpack.config.js
	npm install babel webpack webpack-dev-server --save-dev


`webpack.config.js`

	module.exports = {
	    entry: './main.js',
	    output: {
	        path: './',
	        filename: 'index.js'
	    },
	    devServer: {
	        inline: true,
	        port: 3333
	    },
	    module: {
	        loaders: [
	        {
	            test: /\.js$/,
	            exculde: /node_modules/,
	            loader: 'babel',
	            query: {
	                presets: ['es2015', 'react']
	            }
	        }
	        ]
	    }
	}

`index.html`

	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <title>Setup</title>
	</head>
	<body>
	    <div id="app"></div>
	    <script type="text/javascript" src="index.js"></script>
	</body>
	</html>

`App.js`

	import React from 'react';
	class App extends React.Component {
	    render(){
	        return <div> Hello, World</div>
	    }
	}
	export default App

`main.js`

	import React from 'react';
	import ReactDOM from 'react-dom';
	import App from './App';
	ReactDOM.render(<App />, document.getElementById('app'))

`package.json`

	{
	  "name": "start-react",
	  "version": "1.0.0",
	  "description": "",
	  "main": "index.js",
	  "scripts": {
	    "start": "node node_modules/webpack-dev-server/bin/webpack-dev-server.js"
	  },
	  "author": "",
	  "license": "ISC",
	  "dependencies": {
	    "react": "^15.2.1",
	    "react-dom": "^15.2.1"
	  },
	  "devDependencies": {
	    "babel": "^6.5.2",
	    "babel-core": "^6.10.4",
	    "babel-loader": "^6.2.4",
	    "babel-preset-es2015": "^6.9.0",
	    "babel-preset-react": "^6.11.1",
	    "webpack": "^1.13.1",
	    "webpack-dev-server": "^1.14.1"
	  }
	}
	

	npm start

Visit http://localhost:3333/
