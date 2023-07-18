# ResBook: An Express Node.js App Deployment Guide on Azure with MSSQL
## Introduction : 
ResBook is a comprehensive application for managing restaurant tables and cuisine orders, which is now moving from an in-house infrastructure to the cloud premises in Azure.

## Pre-requisites

- Docker installation on your local machine
- Azure CLI installation on your local machine
- Valid Azure account with an active subscription

## Step 1: Building the Docker Image

Navigate to the directory containing your Dockerfile and execute the following command to create the Docker image:

```bash
docker build -t resbook:v1 .
```

## Step 2: Logging into Azure and Azure Container Registry (ACR)

Authenticate your access to Azure:

```bash
az login
```

Now log into the Azure Container Registry (ACR):

```bash
az acr login --name resbookRN
```

## Step 3: Tagging the Docker Image

Tag the Docker image with the ACR's login server:

```bash
docker tag resbook:v1 resbookrn.azurecr.io/resbook:v1
```

## Step 4: Pushing the Docker Image to ACR

Push your Docker image to the ACR:

```bash
docker push resbookrn.azurecr.io/resbook:v1
```

## Step 5: Creating a Resource Group and MSSQL Server Container on Azure

Initialize a resource group:

```bash
az group create --name ResBookResourceGroup --location eastus
```

Create an Azure Container Instances (ACI) for the MSSQL Server:

```bash
az container create --name mssqlservercontainer --resource-group ResBookResourceGroup --image mcr.microsoft.com/mssql/server:2022-latest --ip-address Public --ports 1433 --cpu 2 --memory 4 --environment-variables ACCEPT_EULA=Y SA_PASSWORD=Admin@12345
```

## Step 6: Deploying the Application Container on Azure

Set up a Web App for Containers service on Azure:

```bash
az appservice plan create --name ResBookServicePlan --resource-group ResBookResourceGroup --sku B1 --is-linux

az webapp create --resource-group ResBookResourceGroup --plan ResBookServicePlan --name resbookapp --deployment-container-image-name resbookrn.azurecr.io/resbook:v1
```

## Step 7: Configuring Advanced Settings 

Navigate to the properties of the web app and in the 'Advanced' tab, include the following JSON configuration:

```json
{
    "name": "DB_HOST",
    "value": "ip_address",
    "slotSetting": false
},
{
    "name": "DB_NAME",
    "value": "resbook",
    "slotSetting": false
},
{
    "name": "DB_PASSWORD",
    "value": "Admin@12345",
    "slotSetting": false
},
{
    "name": "DB_USER",
    "value": "sa",
    "slotSetting": false
}
```

## Step 8: Configuring Environment Variables

Set the necessary environment variables for the application:

```bash
az webapp config appsettings set --resource-group ResBookResourceGroup --name resbookapp --settings DB_HOST=ip_address DB_NAME=resbook DB_PASSWORD=Admin@12345 DB_USER=sa
```

## Step 9: Setting Up The Database

Pull the latest MSSQL Server image from Microsoft Container Registry:

```bash
docker pull mcr.microsoft.com/mssql/server:2022-latest
```

## Step 10: Creating a Database Container

Create a container with the same registry as ResBook:

```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Admin@12345" -p 1433:1433 --name mssqlserver -d mcr.microsoft.com/mssql/server:2022-latest
```

## Step 11: Declaring Environment Variables

Declare the environment variables in the advanced settings of the repository image:

```json
{
    "name": "DB_HOST",
    "value": "db_ip",
    "slotSetting": false
}, 
{
    "name": "DB_NAME",
    "value": "resbook",
    "slotSetting": false
}, 
{
    "name": "DB_PASSWORD",
    "value": "Admin@12345",
    "slotSetting": false
}, 
{
    "name": "DB_USER",
    "value": "sa",
    "slotSetting": false
}
```

## Step 12: Publishing and Creating an Instance From the Image

Publishing the image and creating an instance from it on Azure. 

## Step 13: Configuring Database Schema

Creating `resbook` database and define your `table` and `cuisine` tables:

```sql
-- Database: RestaurantBook
CREATE DATABASE RestaurantBook;
USE RestaurantBook;

-- Table: Restaurant
CREATE TABLE Restaurant
(
    RestaurantID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(100),
    Location NVARCHAR(255),
    ETA INT,
    Rating DECIMAL(3,2),
    Cuisine NVARCHAR(50)
);

-- Table: Cuisine
CREATE TABLE Cuisine
(
    CuisineID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(50)
);

-- Table: RestaurantCuisine
CREATE TABLE RestaurantCuisine
(
    RestaurantID INT,
    CuisineID INT,
    FOREIGN KEY (RestaurantID) REFERENCES Restaurant(RestaurantID),
    FOREIGN KEY (CuisineID) REFERENCES Cuisine(CuisineID),
    PRIMARY KEY (RestaurantID, CuisineID)
);

-- Table: Customer
CREATE TABLE Customer
(
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    Name NVARCHAR(50),
    Email NVARCHAR(255)
);

-- Table: Booking
CREATE TABLE Booking
(
    BookingID INT PRIMARY KEY IDENTITY(1,1),
    RestaurantID INT,
    CustomerID INT,
    BookingTime DATETIME,
    FOREIGN KEY (RestaurantID) REFERENCES Restaurant(RestaurantID),
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
);

```

## Step 14: Inserting Data Into the Tables

Insert data into the `tables` and `cuisines` tables:

```sql
-- Database: RestaurantBook

-- Inserting Data into Restaurant table
INSERT INTO Restaurant (Name, Location, ETA, Rating, Cuisine)
VALUES ('RS-IN', 'Location A', 15, 4.2, 'Indian'),
       ('RS-IT', 'Location B', 20, 4.5, 'Italian'),
       ('RS-MX', 'Location C', 30, 4.7, 'Mexican'),
       ('RS-CH', 'Location D', 25, 4.3, 'Chinese'),
       ('RS-FR', 'Location E', 15, 4.6, 'French'),
       ('RS-USA', 'Location F', 35, 4.0, 'American');

-- Table: Cuisine

-- Inserting Data into Cuisine table
INSERT INTO Cuisine (Name)
VALUES ('Indian'),
       ('Italian'),
       ('Mexican'),
       ('Chinese'),
       ('French'),
       ('American'),
       ('Seafood');

-- Table: RestaurantCuisine
-- Inserting Data into RestaurantCuisine table
-- Assuming each restaurant serves only one type of cuisine
INSERT INTO RestaurantCuisine (RestaurantID, CuisineID)
VALUES (1, 1),
       (2, 2),
       (3, 3),
       (4, 4),
       (5, 5),
       (6, 6),
       (7, 7);

-- Table: Customer

-- Inserting Data into Customer table
INSERT INTO Customer (Name, Email)
VALUES ('Customer A', 'abc@mail.com'),
       ('Customer B', 'test@mail.com'),
       ('Customer C', 'res@mail.com'),
       ('Customer D', 'tin@mail.com');

-- Table: Booking

-- Inserting Data into Booking table
INSERT INTO Booking (RestaurantID, CustomerID, BookingTime)
VALUES (1, 1, GETDATE()),
       (2, 2, DATEADD(HOUR, -1, GETDATE())),
       (3, 3, DATEADD(DAY, -1, GETDATE())),
       (4, 4, DATEADD(DAY, -2, GETDATE()));


## Step 15: Finalizing the Deployment

Finally, open the Azure portal and locate the web application at https://resbook.azurewebsites.net/
