
# Create a streaming Excel custom function
In this lab, you will learn how to create custom functions which perform a simple calculation, request data from the web, and stream real-time data from the web.

## Exercise 1: Create your add-in project
You’ll begin this tutorial by using the Yo Office Yeoman generator, which will automatically populate the files you need for your project.

1. In your command line interface, create a scaffold of your project.  
    
    ```bash
    yo office
    ```
    
    ![Yo Office bash prompts for custom functions](images/yo-office-excel-cfs-stock-ticker.PNG)
    
    Answer the prompts as directed below:  
    - Choose a project type: `Excel Custom Functions Add-in project (Preview: Requires the Insider channel for Excel)`
    - What do you want to name your add-in? `stock-ticker`
    
    After you complete the wizard, the generator will create the project files and install supporting Node components. 

2. From the root folder of your project, start a localhost instance by running the below in the command line:

    ```bash
    npm start
    ```

3. Launch [Excel Online](https://www.office.com/launch/excel). Open a new workbook. 
4. Select **Insert > Add-ins**. Choose **Manage My Add-ins** and select **Upload My Add-in**. Browse for your manifest file, then select **Upload**. 

5. Typically, you need to install a self-signed certificate for your project to work, but this step has been completed for you! 

Now the custom functions in your file will be loaded and ready to use. There are several pre-built functions for you in the Yo Office project. All are attached to a namespace called CONTOSO which is defined in the XML manifest file. Once you start typing `=CONTOSO.` in a cell, the list of available functions will appear.

Let's call `=CONTOSO.ADD42()`. This function adds 42 to any two numbers you provide as arguments. In any cell, type `=CONTOSO.ADD42(1,2)`. It should deliver the answer 45.

_Note that when a call is made in Excel Online, you may see `#GETTING_DATA` appear in a cell. Once a value is returned, this notification should disappear._

## Exercise 2: Create an asynchronous custom function
What if you wanted a function which could fetch and display the price of Microsoft stock that day? Custom functions are designed so you can easily make requests for data from the web asynchronously.
  
You’ll be adding a new function, called `=CONTOSO.STOCKPRICE`, to the **customfunctions.js** file.  The function will take in the name of a stock ticker, such as "MSFT", and return the price of that stock. You'll leverage the IEX Trading API, which is free and does not require authentication.

1. Copy and paste the function below and add it to **customfunctions.js**. 
    
    ```js
    function STOCKPRICE(ticker) {
        return new Promise( 
            function(resolve) {
                let xhr = new XMLHttpRequest();
                let url = "https://api.iextrading.com/1.0/stock/" + ticker + "/price" 
                //add handler for xhr

                xhr.onreadystatechange = function() {
                    if (xhr.readyState == XMLHttpRequest.DONE) {
                    //return result back to Excel

                    resolve(xhr.responseText);
                    }
                }
                //make request

                xhr.open('GET', url, true);
                xhr.send();
        });
    }
    ```
    
    You'll notice in this code that your asynchronous function returns a JavaScript Promise with the data from the IEX Trading API.         Asynchronous custom functions require you to either return a new Promise or use JavaScript's async/await syntax. 

2. In order for Excel to properly run this function, you must also add some metadata to the **./config/customfunctions.json** file.

    ```json
    {
        "name": "STOCKPRICE",
        "description": "Fetches current stock price",
        "helpUrl": "http://dev.office.com",
        "result": {
            "type": "number",
            "dimensionality": "scalar"
        },  
        "parameters": [
            {
                "name": "ticker",
                "description": "stock ticker name",
                "type": "string",
                "dimensionality": "scalar"
            }
        ],
        }
    }
    ```
    You'll notice that this JSON file describes the function, listing the types and dimensionality of the results and parameters.

3. You need to re-upload your manifest for this function to be useable.  In Excel Online, select **Insert > Add-ins**. Choose **Manage My Add-ins** and select **Upload My Add-in**. Browse for your manifest file, then select **Upload**.

4. In any cell of your workbook, enter the function `=CONTOSO.STOCKPRICE("MSFT")`. It should show you the stock price for one share of Microsoft stock right now.

## Exercise 3: Create a streaming asynchronous custom function
The previous function returned the stock price for Microsoft at a particular moment in time, but stock prices are always changing. With custom functions, it is possible to “stream” data from an API to get updates on stock prices in real time.  

To do this, you’ll create a new function, `=CONTOSO.STOCKPRICESTREAM`. It makes a request for updated data every 1000 milliseconds. 

1. Copy and paste the code below into **customfunctions.js**.
    
    ```js
    function STOCKPRICESTREAM(ticker, caller) {

        let result = 0;
        
        setInterval(function() {
            let xhr = new XMLHttpRequest();
            let url = "https://api.iextrading.com/1.0/stock/" + ticker + "/price";
            //add handler for xhr

            xhr.onreadystatechange = function() {
                if (xhr.readyState == XMLHttpRequest.DONE) {
                    //return result back to Excel

                    caller.setResult(xhr.responseText);
                }
            }
            //make request//

            xhr.open('GET', url, true);
            xhr.send();
            }, 1000); //milliseconds
    }
    ```

2. Next, add to the **./config/customfunctions.json** file with the code below.
    
    ```json
    { 
        "name": "STOCKPRICESTREAM",
        "description": "Streams real time stock price",
        "helpUrl": "http://dev.office.com",
        "result": {
            "type": "number",
            "dimensionality": "scalar"
        },  
        "parameters": [
            {
                "name": "ticker",
                "description": "stock ticker name",
                "type": "string",
                "dimensionality": "scalar"
            }
        ],
        "options": {
            "stream": true
        }
    }
    ```
    You'll notice that this JSON file is very similar to the previous function's JSON file, but that a new section has been added for       "options." Because this function is streaming, you must specify this as true in the JSON. 

3. Again, re-upload your manifest for this function to be useable.  In Excel Online, select **Insert > Add-ins**. Choose **Manage My Add-ins** and select **Upload My Add-in**. Browse for your manifest file, then select **Upload**.

4. In any cell in your workbook, run the function `=CONTOSO.STOCKPRICESTREAM("MSFT")`. You do not have to specify the caller because it only serves to hold the callback function, `setResult`, which passes data form the function to Excel to update the cell value. You should receive the current real-time value of Microsoft stock, which will be adjusted every second. 

## Next steps
Congratulations, you’ve completed the custom functions add-in tutorial! This hands-on-lab ends here, but be sure to check out our online docs to learn more about custom functions.

- Learn more about [custom functions on Microsoft Docs](https://docs.microsoft.com/en-us/office/dev/add-ins/excel/custom-functions-overview)