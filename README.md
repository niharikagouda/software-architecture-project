# Details of software-architecture-project

We are getting and reading the data from the dataset provided in the project specification to apply in our prototype,but in real case implementation we are supposed to get the data every 5 minutes from the user and publish the data into a kafka broker node which we launched firstly on our local machine.

Firstly, we take heart disease dataset that we take from UCI machine learning repository. We have chosen processed Cleveland dataset which is used for machine learning purposes. The last column (column -14) is used for predictions. We have producer edge device to publish data to Kafka topics. The producer code will take data from Cleveland dataset and publish to the topics.

Then we have Kafka to csv file to subscribe to the topic data and get the code from there. For simplicity purpose we take 5kb for each csv file, 80 lines make up to 5kb. Its easy to scale up because we have 500kb.

We have to detect how many csv files are created. The Kafka to csv code writes the csv files: This code detects how many files are entering the csv directory. In this code we give azure user account name and account key.

We run the producer code to publish the data in the topic and when we run it we can check how many Kafka to csv files are produced as there is a counter in the Kafka to csv code. We can see that three files are created.

Once the files are uploaded, we can see in Azure, the three files which are created.

Similar way we can keep producing more data as the edge device always emits data. And these files will be detected and created.

Next, we have to train our models. All data is encrypted with Microsoft managed keys. Data sent to the csv files will be trained. We have steps like importing data, preprocessing them and cleaning missed data. Since, we are using preprocessed data hence all the columns are taken for machine learning. If a value is missing, we are deleting the whole row. We are splitting the data 70:30. 70 as training data and 30 as testing data. For training our model, we use multiclass logistic regression. We train, score and evaluate our model. It takes approximately 7 min to train a model.  In scoring, we see the scored probabilities and scored labels. While evaluating we can see the accuracy of the model.

 
We have got 54% accuracy, which means we have trained our model well. We have taken our own model for training and predicting. We are using sklearn and panda libraries. We are taking column 14 for prediction and using binary tree classification as we have only 3 classes namely 0, 1 and 2. We are producing the model to Kafka and using the trained model from the directory for further prediction.
  
Edge device has two functions: one is to deploy the model and predicting the medical status of the patients like 

0-	No heart disease
1-	Moderate heart disease
2-	Rate of high disease is high

Edge device code we deploy the trained model and predict whether the particular patient has heart disease or not.


Code is explained in the folder: https://github.com/niharikagouda/software-architecture-project/blob/master/software_architecture_project.zip in the Code implementation.pdf document.

Demo videos are in the link: https://www.dropbox.com/sh/k8uscsuqkqwsxlo/AAAxU28XY6x_LyCV9ce_vs0va?dl=0





