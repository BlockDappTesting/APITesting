# "Express JS Tutorial - Routers with API"

1. First you may configure ```server.js``` file with requirements
2. After that you may require middleware 
3. Then you configure config file
4. And Router will be configured with respect this
   
Here server file:

```
const express = require('express');
const app = express();
const path = require('path');
const cors = require('cors');
const corsOptions = require('./config/corsOptions');
const { logger } = require('./middleware/logEvents');
const errorHandler = require('./middleware/errorHandler');
const PORT = process.env.PORT || 3500;

// custom middleware logger
app.use(logger);

// built-in middleware to handle urlencoded data
// in other words, form data:  
// ‘content-type: application/x-www-form-urlencoded’
app.use(express.urlencoded({ extended: false }));

// built-in middleware for json 
app.use(express.json());

//serve static files
app.use('/', express.static(path.join(__dirname, '/public')));

// routes
app.use('/', require('./routes/root'));
app.use('/employees', require('./routes/api/employees'));

app.all('*', (req, res) => {
    res.status(404);
    if (req.accepts('html')) {
        res.sendFile(path.join(__dirname, 'views', '404.html'));
    } else if (req.accepts('json')) {
        res.json({ "error": "404 Not Found" });
    } else {
        res.type('txt').send("404 Not Found");
    }
});

app.use(errorHandler);

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));


```


```const corsOptions = require('./config/corsOptions');```

CROS is preety common in nodejs packages. CORS is a node.js package for providing a Connect/Express middleware that can be used to enable CORS with various options.

```
const whitelist = [
    'https://www.yoursite.com',
    'http://127.0.0.1:5500',
    'http://localhost:3500'
];

const corsOptions = {
    origin: (origin, callback) => {
        if (whitelist.indexOf(origin) !== -1 || !origin) {
            callback(null, true)
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    optionsSuccessStatus: 200
}

module.exports = corsOptions;

```

```const { logger } = require('./middleware/logEvents');```

Though express has builtin logger, you would create a logEvents file for traking the requent and respose in servers. Here is the logEvents code. 

```
const { format } = require('date-fns');
const { v4: uuid } = require('uuid');

const fs = require('fs');
const fsPromises = require('fs').promises;
const path = require('path');

const logEvents = async (message, logName) => {
    const dateTime = `${format(new Date(), 'yyyyMMdd\tHH:mm:ss')}`;
    const logItem = `${dateTime}\t${uuid()}\t${message}\n`;

    try {
        if (!fs.existsSync(path.join(__dirname, '..', 'logs'))) {
            await fsPromises.mkdir(path.join(__dirname, '..', 'logs'));
        }

        await fsPromises.appendFile(path.join(__dirname, '..', 'logs', logName), logItem);
    } catch (err) {
        console.log(err);
    }
}

const logger = (req, res, next) => {
    logEvents(`${req.method}\t${req.headers.origin}\t${req.url}`, 'reqLog.txt');
    console.log(`${req.method} ${req.path}`);
    next();
}

module.exports = { logger, logEvents };

```

```const errorHandler = require('./middleware/errorHandler');```

## ErrorHandling is the key part of server request

We may use this code for handling error request and it will provide meaningful response. 

```
const { logEvents } = require('./logEvents');

const errorHandler = (err, req, res, next) => {
    logEvents(`${err.name}: ${err.message}`, 'errLog.txt');
    console.error(err.stack)
    res.status(500).send(err.message);
}

module.exports = errorHandler;

```

## API

An Express application is most often used as a backend application in a client-server architecture whereas the client could be written in React.js or another popular frontend solution and the server could be written in Express. Both entities result in a client-server architecture (frontend and backend relationship) whereas the backend would be needed for (A) business logic that shouldn't be exposed as source code to the frontend application -- otherwise it would be accessible in the browser -- or for (B) establishing connections to third-party data sources (e.g. database(s)).


Here we use employee json file for testing API

```
const data = {
    employees: require('../model/employees.json'),
    setEmployees: function (data) { this.employees = data }
}

const getAllEmployees = (req, res) => {
    res.json(data.employees);
}

const createNewEmployee = (req, res) => {
    const newEmployee = {
        id: data.employees?.length ? data.employees[data.employees.length - 1].id + 1 : 1,
        firstname: req.body.firstname,
        lastname: req.body.lastname
    }

    if (!newEmployee.firstname || !newEmployee.lastname) {
        return res.status(400).json({ 'message': 'First and last names are required.' });
    }

    data.setEmployees([...data.employees, newEmployee]);
    res.status(201).json(data.employees);
}

const updateEmployee = (req, res) => {
    const employee = data.employees.find(emp => emp.id === parseInt(req.body.id));
    if (!employee) {
        return res.status(400).json({ "message": `Employee ID ${req.body.id} not found` });
    }
    if (req.body.firstname) employee.firstname = req.body.firstname;
    if (req.body.lastname) employee.lastname = req.body.lastname;
    const filteredArray = data.employees.filter(emp => emp.id !== parseInt(req.body.id));
    const unsortedArray = [...filteredArray, employee];
    data.setEmployees(unsortedArray.sort((a, b) => a.id > b.id ? 1 : a.id < b.id ? -1 : 0));
    res.json(data.employees);
}

const deleteEmployee = (req, res) => {
    const employee = data.employees.find(emp => emp.id === parseInt(req.body.id));
    if (!employee) {
        return res.status(400).json({ "message": `Employee ID ${req.body.id} not found` });
    }
    const filteredArray = data.employees.filter(emp => emp.id !== parseInt(req.body.id));
    data.setEmployees([...filteredArray]);
    res.json(data.employees);
}

const getEmployee = (req, res) => {
    const employee = data.employees.find(emp => emp.id === parseInt(req.params.id));
    if (!employee) {
        return res.status(400).json({ "message": `Employee ID ${req.params.id} not found` });
    }
    res.json(employee);
}

module.exports = {
    getAllEmployees,
    createNewEmployee,
    updateEmployee,
    deleteEmployee,
    getEmployee
}

```







