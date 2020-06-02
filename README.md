# software-architecture-project
Details of our software architecture project
Software architecture project

We are getting and reading the data from the dataset provided in the project specification to apply in our prototype,but in real case implementation we are supposed to get the data every 5 minutes from the user and publish the data into a kafka broker node which we launched firstly on our local machine.

Firstly, we take heart disease dataset that we take from UCI machine learning repository. We have chosen processed Cleveland dataset which is used for machine learning purposes. The last column (column -14) is used for predictions. We have producer edge device to publish data to Kafka topics. The producer code will take data from Cleveland dataset and publish to the topics.

1.	####Producer code  
2.	  
3.	from kafka import KafkaProducer  
4.	from kafka.errors import KafkaError  
5.	from kafka import KafkaConsumer  
6.	import numpy as np  
7.	  
8.	producer = KafkaProducer(bootstrap_servers=['localhost:9092'])  
9.	  
10.	file_object  = open("processed.cleveland.data", "r")  
11.	accumulate=""  
12.	tag=0  
13.	for line in file_object:  
14.	    for value in line.split(','):  
15.	            accumulate=accumulate + " " + value  
16.	    print(accumulate)  
17.	    producer.send('data',  bytearray(accumulate, 'utf-8'),key=bytearray(str(tag), 'utf-8'))  
18.	      
19.	    accumulate=""  
20.	    tag+=1  
21.	    producer.flush()  

Then we have Kafka to csv file to subscribe to the topic data and get the code from there. For simplicity purpose we take 5kb for each csv file, 80 lines make up to 5kb. Its easy to scale up because we have 500kb.

1.	####Kafka to csv file  
2.	  
3.	from kafka import KafkaProducer  
4.	from kafka.errors import KafkaError  
5.	from kafka import KafkaConsumer  
6.	import csv  
7.	  
8.	consumer = KafkaConsumer('data',  
9.	                         group_id='0',  
10.	                         bootstrap_servers=['localhost:9092'])  
11.	  
12.	flag=0  
13.	while True:  
14.	    print(flag)  
15.	    counter=0  
16.	    file= open('csv_directory/Data'+str(flag)+'.csv', 'w', newline='')  
17.	    writer = csv.writer(file)  
18.	    writer.writerow(["Column1", "Column2", "Column3", "Column4", "Column5", "Column6", "Column7", "Column8", "Column9", "Column10", "Column11", "Column12", "Column13", "Column14"])  
19.	    consumer = KafkaConsumer('data')  
20.	    for msg in consumer:  
21.	        if counter <= 80:  
22.	            array=[]  
23.	            for x in  range(1,14):  
24.	                array.append(msg[6].decode('utf-8').split(' ')[x])  
25.	            array.append(msg[6].decode('utf-8').split(' ')[14][0])  
26.	#             print(msg[6].decode('utf-8').split(' ')[14][0])  
27.	            writer.writerow(array)  
28.	            counter = counter+1  
29.	        else:  
30.	            array=[]  
31.	            for x in  range(1,14):  
32.	                array.append(msg[6].decode('utf-8').split(' ')[x])  
33.	            array.append(msg[6].decode('utf-8').split(' ')[14][0])  
34.	#             print(msg[6].decode('utf-8').split(' ')[14][0])  
35.	            writer.writerow(array)  
36.	            counter = counter+1  
37.	        if counter==80:  
38.	            flag=flag+1  
39.	            counter=0  
40.	            file.close()  
41.	            break;


We have to detect how many csv files are created. The Kafka to csv code writes the csv files: This code detects how many files are entering the csv directory. In this code we give azure user account name and account key.


1.	####Detect csv files  
2.	  
3.	import  uuid, sys  
4.	from azure.storage.blob import BlockBlobService, PublicAccess  
5.	import os  
6.	import win32file  
7.	import win32con  
8.	import time  
9.	  
10.	def run_sample(file):  
11.	    try:  
12.	        # Create the BlockBlockService that is used to call the Blob service for the storage account  
13.	        block_blob_service = BlockBlobService(account_name='jayasurya217', account_key='80YxeeJDc5tf1hnl/CgenV8U4/4I1n1NCJmeV5Fe3mLVL4JZaT7Nr3jyvMGZdz+EhA0tEdjANzY2sjrbT8FCyA==')  
14.	  
15.	        # Create a container called 'quickstartblobs'.  
16.	        container_name ='project'  
17.	        block_blob_service.create_container(container_name)  
18.	        # Set the permission so the blobs are public.  
19.	        block_blob_service.set_container_acl(container_name, public_access=PublicAccess.Container)  
20.	  
21.	        # Create a file in Documents to test the upload and download.  
22.	        local_path=os.path.abspath(os.path.curdir)  
23.	        local_file_name = "csv_directory/"+file  
24.	        full_path_to_file =os.path.join(local_path, local_file_name)  
25.	  
26.	        # Write text to the file.  
27.	        #file = open(full_path_to_file,  'w')  
28.	        #file.write("Hello, World!")  
29.	        #file.close()  
30.	  
31.	        print("Temp file = " + full_path_to_file)  
32.	        print("\nUploading to Blob storage as blob" + local_file_name)  
33.	  
34.	        # Upload the created file, use local_file_name for the blob name  
35.	        block_blob_service.create_blob_from_path(container_name, local_file_name, full_path_to_file)  
36.	  
37.	        # List the blobs in the container  
38.	        print("\nList blobs in the container")  
39.	        generator = block_blob_service.list_blobs(container_name)  
40.	        for blob in generator:  
41.	            print("\t Blob name: " + blob.name)  
42.	  
43.	        sys.stdout.flush()  
44.	  
45.	    except Exception as e:  
46.	        print(e)  
47.	  
48.	  
49.	ACTIONS = {  
50.	  1 : "Created",  
51.	  2 : "Deleted",  
52.	  3 : "Updated",  
53.	  4 : "Renamed from something",  
54.	  5 : "Renamed to something"  
55.	}  
56.	# Thanks to Claudio Grondi for the correct set of numbers  
57.	FILE_LIST_DIRECTORY = 0x0001  
58.	  
59.	path_to_watch = "./csv_directory"  
60.	hDir = win32file.CreateFile (  
61.	  path_to_watch,  
62.	  FILE_LIST_DIRECTORY,  
63.	  win32con.FILE_SHARE_READ | win32con.FILE_SHARE_WRITE | win32con.FILE_SHARE_DELETE,  
64.	  None,  
65.	  win32con.OPEN_EXISTING,  
66.	  win32con.FILE_FLAG_BACKUP_SEMANTICS,  
67.	  None  
68.	)  
69.	while 1:  
70.	  #  
71.	  # ReadDirectoryChangesW takes a previously-created  
72.	  # handle to a directory, a buffer size for results,  
73.	  # a flag to indicate whether to watch subtrees and  
74.	  # a filter of what changes to notify.  
75.	  #  
76.	  # NB Tim Juchcinski reports that he needed to up  
77.	  # the buffer size to be sure of picking up all  
78.	  # events when a large number of files were  
79.	  # deleted at once.  
80.	  #  
81.	  results = win32file.ReadDirectoryChangesW (  
82.	    hDir,  
83.	    1024,  
84.	    True,  
85.	    win32con.FILE_NOTIFY_CHANGE_FILE_NAME |  
86.	     win32con.FILE_NOTIFY_CHANGE_DIR_NAME |  
87.	     win32con.FILE_NOTIFY_CHANGE_ATTRIBUTES |  
88.	     win32con.FILE_NOTIFY_CHANGE_SIZE |  
89.	     win32con.FILE_NOTIFY_CHANGE_LAST_WRITE |  
90.	     win32con.FILE_NOTIFY_CHANGE_SECURITY,  
91.	    None,  
92.	    None  
93.	  )  
94.	  for action, file in results:  
95.	    full_filename = os.path.join (path_to_watch, file)  
96.	    if full_filename.endswith('.csv') and os.path.getsize("csv_directory/"+file)>0 :  
97.	        print (full_filename+ ACTIONS.get (action, "Uploading..."))  
98.	        run_sample(file)  

We run the producer code to publish the data in the topic and when we run it we can check how many Kafka to csv files are produced as there is a counter in the Kafka to csv code. We can see that three files are created.

Once the files are uploaded, we can see in Azure, the three files which are created.

Similar way we can keep producing more data as the edge device always emits data. And these files will be detected and created.

Next, we have to train our models. All data is encrypted with Microsoft managed keys. Data sent to the csv files will be trained. We have steps like importing data, preprocessing them and cleaning missed data. Since, we are using preprocessed data hence all the columns are taken for machine learning. If a value is missing, we are deleting the whole row. We are splitting the data 70:30. 70 as training data and 30 as testing data. For training our model, we use multiclass logistic regression. We train, score and evaluate our model. It takes approximately 7 min to train a model.  In scoring, we see the scored probabilities and scored labels. While evaluating we can see the accuracy of the model.

 
We have got 54% accuracy, which means we have trained our model well. We have taken our own model for training and predicting. We are using sklearn and panda libraries. We are taking column 14 for prediction and using binary tree classification as we have only 3 classes namely 0, 1 and 2. We are producing the model to Kafka and using the trained model from the directory for further prediction.
  
1.	####Predict model code  
2.	  
3.	# import pandas  
4.	import pandas as pd  
5.	from sklearn.model_selection import train_test_split  
6.	from xgboost import XGBClassifier  
7.	from sklearn.metrics import roc_auc_score  
8.	from sklearn import preprocessing  
9.	from sklearn import tree  
10.	from kafka import KafkaProducer  
11.	from kafka.errors import KafkaError  
12.	from joblib import dump, load  
13.	import numpy as np  
14.	counter=0  
15.	  
16.	  
17.	df = pd.read_csv('./csv_directory/Data'+str(counter)+'.csv')  
18.	df.head()  
19.	print(df.shape)  
20.	df.replace(to_replace = 7.0,   
21.	                 value =3.0)  
22.	df.astype({'Column13': 'float'}).dtypes  
23.	# df.astype({'Column12': 'float'}).dtypes  
24.	num_of_classes = len(df.Column14.unique())  
25.	print(num_of_classes)  
26.	  
27.	# split train input and output data  
28.	X = df.drop(axis=0, columns=['Column12','Column14'])  
29.	Y = df.Column14  
30.	  
31.	#Print the shape of X and Y  
32.	print(X.shape)  
33.	print(Y.shape)  
34.	  
35.	# Split into training and test sets  
36.	X_train, X_test, y_train, y_test = train_test_split(X, Y, test_size=0.30, random_state=42)  
37.	  
38.	# Create a classifier  
39.	xgb = XGBClassifier(booster='gbtree', objective='multi:softprob', random_state=42, eval_metric="auc", num_class=num_of_classes)  
40.	  
41.	# Fit the classifier with the training data  
42.	xgb.fit(X_train,y_train)  
43.	# Use trained model to predict output of test dataset  
44.	val = xgb.predict(X_test)  
45.	  
46.	lb = preprocessing.LabelBinarizer()  
47.	lb.fit(y_test)  
48.	  
49.	y_test_lb = lb.transform(y_test)  
50.	val_lb = lb.transform(val)  
51.	  
52.	roc_auc_score(y_test_lb, val_lb, average='macro')  
53.	  
54.	dump(xgb, './trained_models/model'+str(counter)+'.joblib')   
55.	producer = KafkaProducer(bootstrap_servers=['localhost:9092'])  
56.	producer.send('trained_model',  bytearray('./trained_models/model'+str(counter)+'.joblib', 'utf-8'))  
57.	counter+=1  

Edge device has two functions: one is to deploy the model and predicting the medical status of the patients like 

0-	No heart disease
1-	Moderate heart disease
2-	Rate of high disease is high

Edge device code we deploy the trained model and predict whether the particular patient has heart disease or not.


1.	####Edge_device_deploy_code  
2.	  
3.	from kafka import KafkaProducer  
4.	from kafka.errors import KafkaError  
5.	from kafka import KafkaConsumer  
6.	from joblib import dump, load  
7.	import pandas as pd  
8.	from sklearn.model_selection import train_test_split  
9.	from xgboost import XGBClassifier  
10.	from sklearn.metrics import roc_auc_score  
11.	from sklearn import preprocessing  
12.	  
13.	def deploy_model(model_name):  
14.	    prediction_model = load(model_name)  
15.	def predict_medical_status(patient_data):  
16.	    numpy_patient_data = pd.DataFrame({"Column1":[patient_data[0]],"Column2":[patient_data[1]],"Column3":[patient_data[2]],"Column4":[patient_data[3]],  
17.	                        "Column5":[patient_data[4]],"Column6":[patient_data[5]],"Column7":[patient_data[6]],"Column8":[patient_data[7]],  
18.	                        "Column9":[patient_data[8]],"Column10":[patient_data[9]],"Column11":[patient_data[10]],"Column13":[patient_data[11]]})  
19.	    print(prediction_model.predict(numpy_patient_data))  
20.	  
21.	prediction_model=load("./trained_models/model0.joblib")  
22.	consumer = KafkaConsumer('trained_model',  
23.	                         group_id='my-group',  
24.	                         bootstrap_servers=['localhost:9092'])  
25.	  
26.	for message in consumer:  
27.	    print(message)   
28.	    print(message[6].decode('utf-8'))   
29.	    timestamp = message[3]  
30.	    model_name= message[6].decode('utf-8')  
31.	    deploy_model(model_name)  
32.	  
33.	predict_medical_status([67,1,4,160,286,0,2,108,1,1.5,2,3])  
34.	  
35.	prediction_model=load("./trained_models/model0.joblib")  
36.	  
37.	prediction_model.predict(x)  
38.	  
39.	prediction_model = load(prediction_model)  

1.	###Download models  
2.	  
3.	----------------------------------------------------------------------------------  
4.	# MIT License  
5.	#  
6.	# Copyright(c) Microsoft Corporation. All rights reserved.  
7.	#  
8.	# Permission is hereby granted, free of charge, to any person obtaining a copy  
9.	# of this software andassociated documentation files (the "Software"), to deal  
10.	# in the Software without restriction, including without limitation the rights  
11.	# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell  
12.	# copies of the Software, and to permit persons to whom the Software is  
13.	# furnished to do so, subject to the following conditions:  
14.	# ----------------------------------------------------------------------------------  
15.	# The above copyright notice and this permission notice shall be included in all  
16.	# copies or substantial portions of the Software.  
17.	#  
18.	# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR  
19.	# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,  
20.	# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE  
21.	# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER  
22.	# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,  
23.	# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE  
24.	# SOFTWARE.  
25.	  
26.	  
27.	  
28.	import  uuid, sys  
29.	from azure.storage.blob import BlockBlobService, PublicAccess  
30.	  
31.	# ---------------------------------------------------------------------------------------------------------  
32.	# Method that creates a test file in the 'Documents' folder.  
33.	# This sample application creates a test file, uploads the test file to the Blob storage,  
34.	# lists the blobs in the container, and downloads the file with a new name.  
35.	# ---------------------------------------------------------------------------------------------------------  
36.	# Documentation References:  
37.	# Associated Article - https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-python  
38.	# What is a Storage Account - http://azure.microsoft.com/en-us/documentation/articles/storage-whatis-account/  
39.	# Getting Started with Blobs-https://docs.microsoft.com/en-us/azure/storage/blobs/storage-python-how-to-use-blob-storage  
40.	# Blob Service Concepts - http://msdn.microsoft.com/en-us/library/dd179376.aspx  
41.	# Blob Service REST API - http://msdn.microsoft.com/en-us/library/dd135733.aspx  
42.	# ----------------------------------------------------------------------------------------------------------  
43.	  
44.	import os  
45.	  
46.	import win32file  
47.	import win32con  
48.	  
49.	run_sample():  
50.	    try:  
51.	        # Create the BlockBlockService that is used to call the Blob service for the storage account  
52.	        block_blob_service = BlockBlobService(account_name='jayasuria217', account_key='wXL7djNzWGUZOG/kwDXkabGDLkQ29Z0eDvNMh/n1eDo61My9HzWh1Ilyi/qzJjFbDTo2lUX1pDgXmZeeOZxa0g==')  
53.	  
54.	        # Create a container called 'quickstartblobs'.  
55.	        container_name ='azureml-blobstore-26dc84d7-bebf-46e7-a16c-adf7ddaade26'  
56.	  
57.	  
58.	  
59.	        # Create a file in Documents to test the upload and download.  
60.	        local_path=os.path.abspath(os.path.curdir)  
61.	        local_file_name = "./azureml/"  
62.	        full_path_to_file =os.path.join(local_path, local_file_name)  
63.	  
64.	        # List the blobs in the container  
65.	        print("\nList blobs in the container")  
66.	        generator = block_blob_service.list_blobs(container_name)  
67.	        for blob in generator:  
68.	            print("\t Blob name: " + blob.name)  
69.	  
70.	        # Download the blob(s).  
71.	        # Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.  
72.	        print("\nDownloading blob to " + full_path_to_file)  
73.	        block_blob_service.get_blob_to_path(container_name, local_file_name, full_path_to_file)  
74.	  
75.	        sys.stdout.write("Sample finished running. When you hit <any key>, the sample will be deleted and the sample "  
76.	                         "application will exit.")  
77.	        sys.stdout.flush()  
78.	        input()  
79.	                                      
80.	    except Exception as e:  
81.	        print(e)  
82.	  
83.	run_sample() 



