# Create a ML model

1) create model.ipynb file in jupyter notebook, for building a ml model we need following libraries
    -import numpy as np
    -import pandas as pd
    -import seaborn as sns
    -import matplotlib.pyplot as plt
2) perform missing value imputation, cleaning outliers, etc.,

3) after analysing and cleaning data, split the data into 80:20 (80% for train data, 20% as test data)

4) build and train different models and choose the model with highest accuracy score

5) now dump the model using pickle library
    -import pickle
    -pickle.dump(lm_r, open('model.pkl','wb'))

6) running the above code will create model.pkl file

7) now create a desired directory and add model.ipynb , model.pkl files in it
   example: /predictapp
            |
            ---/flaskapi
               |
               ---model.ipynb
               ---model.pkl


# Creating flask api

1) create app.py file, for flask api we need the following libraries
    -from flask import Flask, request, jsonify
    -import pickle
    -import numpy as np
    -import pandas as pd
    -from sklearn.linear_model import Ridge

2) @app.route('/predict', methods=['POST'])
    -/predict: This defines the URL path for the endpoint. When someone accesses
    -methods=['POST']: This specifies that this endpoint should only accept POST requests.

3) now load the pickle file and map the values from json 

4) use the following port to run the server
    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)


# testing before making as container in docker

1) using postman test the flask api
 
2) in postman application use POST and in the url tab use the folloing url
    -http://localhost:5000/predict
    
    -make sure that flask is running otherise it wont work

3) now use json to get the output if you get your output go for docker
    -input
    {
        "rooms": 3,
        "bathrooms": 2,
        "square_feet": 1200,
        "location": "downtown"
    } 
    -output
    {
        "predicted_price": 23375580.725906506
    }  


# Creating docker image using dockerfile

1) install and run docker desktop

2) now create a Dockerfile to create an image which contains following commands

    # Use an official Python runtime as a parent image
    FROM python:3.9

    # Set the working directory in the container
    WORKDIR /app

    # Copy requirements
    COPY requirements.txt .

    # Install any needed packages specified in requirements.txt
    RUN pip install --upgrade pip
    RUN pip install -r requirements.txt

    COPY . .

    # Expose port 5000 for the Flask app
    EXPOSE 5000

    # Define environment variable
    ENV FLASK_APP=app.py

    # Run the Flask app when the container launches
    CMD ["flask", "run", "--host=0.0.0.0","--port=5000"]

3) now create docker-compose.yml file which contains

  services:
    flask_ml:
        container_name: flaskapi
        image: predictapp/flaskapi
        build: ./flaskapi  # Build the Flask app in the api folder
        ports:
            - "5000:5000"  # Map container port 5000 to localhost 5000
        volumes:
            - ./flaskapi:/app   # Mount the current directory to /app in the container
        environment:
            - FLASK_ENV=development

4) now use following command to login to docker
    -docker login

5) to clear the docker cache without any further issues use the following command
    -docker system prune --all

6) now use the following command to create image and the container and to start the server
    -docker-compose up --build

7) now use the post man again to test and you will get it.