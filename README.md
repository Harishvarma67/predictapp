# Create a ML model

Note : the path for copy ames housing data set is : `'E:/predictapp/flaskapi/AmesHousing.csv'` you can change this as per your requirements<br />

The path for the entire project look like this<br />

`/predictapp` <br />
`|` <br /> 
`----->/flaskapi` <br />
`|     |` <br />
`|     ----->AmesHousing.csv `<br /> 
`|     ----->app.py `<br />
`|     ----->Dockerfile`  <br />
`|     ----->model.ipynb`  <br />
`|     ----->model.plk`  <br />
`|     ----->requirements.txt` <br /> 
`|` <br />
`----->docker-compose.yml` <br />
`----->README.md` <br />


1) create model.ipynb file in jupyter notebook, for building a ml model we need following libraries <br />
    `-import numpy as np` <br />
    `-import pandas as pd` <br />
    `-import seaborn as sns` <br />
    `-import matplotlib.pyplot as plt` <br />
2) perform missing value imputation, cleaning outliers, etc.,<br />

3) after analysing and cleaning data, split the data into 80:20 (80% for train data, 20% as test data)<br />

4) build and train different models and choose the model with highest accuracy score<br />

5) now dump the model using pickle library <br />
    -`import pickle` <br />
    -`pickle.dump(lm_r, open('model.pkl','wb'))`<br />

6) running the above code will create model.pkl file<br />

7) now create a desired directory and add model.ipynb , model.pkl files in it example: <br />
   
            `/predictapp` <br />
            `| `<br />
            `----->/flaskapi` <br />
            `      |` <br />
            `      ----->model.ipynb` <br />
            `      ----->model.pkl` <br />


# Creating flask api

1) create app.py file, for flask api we need the following libraries <br />
    `-from flask import Flask, request, jsonify` <br />
    `-import pickle` <br />
    `-import numpy as np` <br />
    `-import pandas as pd` <br />
    `-from sklearn.linear_model import Ridge` <br />

2) `@app.route('/predict', methods=['POST'])` <br />
    -/predict: This defines the URL path for the endpoint. When someone accesses <br />
    -methods=['POST']: This specifies that this endpoint should only accept POST requests. <br />

3) now load the pickle file and map the values from json <br />
    -`with open('model.pkl', 'rb') as model_file:`  <br />
     `     lm_r = pickle.load(model_file)` <br />

5) use the following port to run the server <br />
    `if __name__ == '__main__':` <br />
    `   app.run(host='0.0.0.0', port=5000)` <br />


# testing before making as container in docker

1) using postman test the flask api<br />
 
2) in postman application use POST and in the url tab use the folloing url <br />
    `http://localhost:5000/predict` <br />
    
    -make sure that flask is running otherise it wont work <br />

3) now use json to get the output if you get your output go for docker<br />
    `-input` <br />
    `{` <br />
    `    "rooms": 3,` <br />
    `    "bathrooms": 2,` <br />
    `    "square_feet": 1200,` <br />
    `    "location": "downtown"` <br />
    `}`  <br />
    `-output` <br />
    `{` <br />
    `    "predicted_price": 23375580.725906506` <br />
    `}`  <br />


# Creating docker image using dockerfile

1) install and run docker desktop<br />

2) now create a Dockerfile to create an image which contains following commands<br />

    `FROM python:3.9 `# Use an official Python runtime as a parent image<br />
   
    `WORKDIR /app` # Set the working directory in the container<br />

    `COPY requirements.txt . `# Copy requirements<br />

    `RUN pip install --upgrade pip`  # Install any needed packages specified in requirements.txt<br />
    `RUN pip install -r requirements.txt`<br />

    `COPY . .`<br />

    `EXPOSE 5000` # Expose port 5000 for the Flask app<br />

    `ENV FLASK_APP=app.py` # Define environment variable<br />

    `CMD ["flask", "run", "--host=0.0.0.0","--port=5000"] `   # Run the Flask app when the container launches<br />

4) now create docker-compose.yml file which contains<br />

  `services:` <br />
  `    flask_ml: `<br />
  `        container_name: flaskapi` <br />
  `         image: predictapp/flaskapi` <br />
  `         build: ./flaskapi ` # Build the Flask app in the api folder <br />
  `         ports:` <br />
  `           - "5000:5000"`  # Map container port 5000 to localhost 5000 <br />
  `         volumes: `<br />
  `           - ./flaskapi:/app`   # Mount the current directory to /app in the container <br />
  `         environment:` <br />
  `           - FLASK_ENV=development` <br />

4) now use following command to login to docker <br />
    `-docker login `<br />

5) to clear the docker cache without any further issues use the following command <br />
    `-docker system prune --all` <br />

6) now use the following command to create image and the container and to start the server <br />
    `-docker-compose up --build` <br />

7) now use the postman again use the following server to test and you will get it.<br />
    `http://localhost:5000/predict`
