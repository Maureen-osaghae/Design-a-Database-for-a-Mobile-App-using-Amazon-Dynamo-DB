#  Design and Implementation of a DynamoDB-Based Mobile Application Backend

---

##  1. Introduction

In today’s digital era, social media applications rely heavily on scalable, low-latency databases to manage large volumes of user-generated content.  
This project demonstrates how to design and implement a **mobile social networking application backend** using **Amazon DynamoDB**, a fully managed NoSQL database service by AWS.

The system supports features such as:
- User profiles
- Photo uploads
- Reactions (likes, etc.)
- Following relationships

It uses **DynamoDB’s single-table design** for optimized performance and scalability.

---

##  2. Project Objectives

- Design a scalable, single-table DynamoDB schema for a social networking app.  
- Model entities and relationships efficiently.  
- Implement access patterns for users, photos, reactions, and friendships.  
- Use AWS services including **Cloud9**, **DynamoDB**, and **boto3** (Python SDK).  
- Implement **secondary indexes** and **transactions** for advanced query and data integrity.

---

##  3. Tools and Technologies

| Tool | Description |
|------|--------------|
| **Amazon Web Services (AWS)** | Cloud platform used for deployment |
| **Amazon DynamoDB** | NoSQL database used for data storage |
| **AWS Cloud9 IDE** | Cloud-based integrated development environment |
| **Python 3** | Programming language used for scripting |
| **boto3** | AWS SDK for Python |
| **AWS CLI** | Command line interface for managing AWS services |

---

##  4. System Design

## Entities
- **User**
- **Photo**
- **Reaction**
- **Friendship**

## Relationships
- A **User** can have multiple **Photos**.
- A **Photo** can have multiple **Reactions**.
- A **Friendship** represents a one-way follow relationship between users.

---

##  5. Primary Key Schema

| Entity | Partition Key (PK) | Sort Key (SK) |
|--------|---------------------|---------------|
| **User** | `USER#<USERNAME>` | `#METADATA#<USERNAME>` |
| **Photo** | `USER#<USERNAME>` | `PHOTO#<USERNAME>#<TIMESTAMP>` |
| **Reaction** | `REACTION#<USERNAME>#<TYPE>` | `PHOTO#<USERNAME>#<TIMESTAMP>` |
| **Friendship** | `USER#<USERNAME>` | `#FRIEND#<FRIEND_USERNAME>` |

This composite key design enables efficient data access across all entities in a **single DynamoDB table**.

---

## 6. Implementation Steps

### **Step 1: Setup AWS Cloud9**

1. Go to **AWS Cloud9** in your AWS console.  
2. Create an environment named `DynamoDB Quick Photos`.  
3. Use default settings and launch.

   <img width="955" height="305" alt="image" src="https://github.com/user-attachments/assets/5a38db68-8048-44d0-a50e-200d89b3d187" />

<img width="959" height="422" alt="image" src="https://github.com/user-attachments/assets/383bce03-ebab-401c-b41f-428307c53d82" />

 Choose Next step > Select Create environment 

 <img width="959" height="234" alt="image" src="https://github.com/user-attachments/assets/beee9ef5-5183-43ee-8d5e-9432532c6596" />

 Install Python and dependencies:

      sudo yum install python3 -y
      pip3 install boto3

## 7. Step 2: Download supporting code

     cd ~/environment
      curl -o quick-photos.tar https://s3.amazonaws.com/ddb-labs/quick-photos.tar
      tar -xvf quick-photos.tar

<img width="690" height="395" alt="image" src="https://github.com/user-attachments/assets/969f625f-106d-4453-a86c-5d4824e1e916" />

Run the ls command ls
 
 ● There should be two directories now, application and scripts

 <img width="623" height="73" alt="image" src="https://github.com/user-attachments/assets/146d93bb-4826-43e5-9d77-e0ba4a686fae" />

 Set permissions:

      chmod +x ~/environment/scripts/*.py


The first step of any data modeling exercise is to build a diagram to show the entities in your
application and how they relate to each other.

In our application, we have the following entities:
● User
● Photo
● Reaction
● Friendship

A User can have many Photos, and a Photo can have many Reactions. Finally, the Friendship
entity represents a many-to-many relationship between Users, as a User can follow multiple
Users and be followed by multiple other Users.
With these entities and relationships in mind, our entity-relationship diagram is shown below.

<img width="512" height="296" alt="image" src="https://github.com/user-attachments/assets/f85661bd-8df3-4c56-b4f7-bc8199bb393b" />

Consider user profile access patterns
Now that we have our entity-relationship diagram, consider the access patterns around our
entities. Let’s start with users.

The users of our mobile application will need to create user profiles. These profiles will include
information such as a username, profile picture, location, current status, and interests for a
given user.
Users will be able to browse the profile of other users. A user may want to browse the profile of
another user to see if the user is interesting to follow or simply to read some background on an
existing friend 

Over time, a user will want to update their profile to display a new status or to update their
interests as they change.
Based on this information, we have three access patterns:
- Create user profile (Write)
- Update user profile (Write)
-  Get user profile (Read)

 ## Consider photo access patterns

Now, let’s look at the access patterns around photos.
Our mobile application allows users to upload and share photos with their friends, similar to
Instagram or Snapchat. When users upload a photo, you will need to store information such as
the time the photo was uploaded and the location of the file on your content delivery network
(CDN).
When users aren’t uploading photos, they will want to browse photos of their friends. If they visit
a friend’s profile, they should see the photos for a user with the most recent photos showing
first. If they really like a photo, they can ‘react’ to the photo using one of four predefined
reactions -- a heart, a smiley face, a thumbs up, or a pair of sunglasses. Viewing a photo should
display the current reactions for the photo.
In this section, we have the following access patterns:
- Upload photo for user (Write)
- View recent photos for user (Read)
- React to a photo (Write)
- View photo and reactions (Read)

  ## Friendship access patterns
Finally, let’s consider the access patterns around friendship.
Many popular mobile applications have a social network aspect. 

You can follow friends, view updates on your friends’ activities, and receive recommendations on other friends you may want to follow.
In your application, a friendship is a one-way relationship, like Twitter. One user can choose to
follow another user, and that user may choose to follow the user back. For our application, we
will call the users that follow a user “followers”, and we will call the users that a user is following
the “followed”.
Based on this information, we have the following access patterns:
- Follow user (Write)
- View followers for user (Read)
- View followed for user (Read)

 ## Design the primary key
Let’s consider the different entities, as suggested in the preceding introduction. In the mobile
application, we have the following entities:
● Users
● Photos
● Reactions
● Friendship

These entities show three different kinds of data relationships.
First, each user on your application will have a single user profile represented by a User entity in
your table.
Next, a user will have multiple photos represented in your application, and a photo will have
Multiple reactions. These are both one-to-many relationships.
Finally, the Friendship entity is a representation of a many-to-many relationship. The Friendship
entity represents when one user is following another user in your application. 

It is a many-to-many relationship as one user may follow multiple other users, and a user may have multiple followers.
Having a many-to-many mapping is usually an indication that you will want to satisfy two Query
patterns, and our application is no exception. On the Friendship entity, we have an access
pattern that needs to find all users that follow a particular user as well as an access pattern to
find all of the users that a given user follows.

Because of this, we’ll use a composite primary key with both a HASH and RANGE value. The
composite primary key will give us the Query ability on the HASH key to satisfy one of the query
patterns we need. 

In the DynamoDB API specification, the partition key is called HASH and the
sort key is called RANGE, and in this guide we will use the API terminology interchangeably and
especially when we discuss the code or DynamoDB JSON wire format.

Note that the one-to-one entity -- User -- doesn’t have a natural property for the RANGE value.
Because it’s a one-to-one mapping, the access patterns will be a basic key-value lookup. Since
your table design requires a RANGE property, you can provide a filler value for the RANGE key.

First, for the User entity, the HASH value will be USER#<USERNAME>. Notice that you’re
using a prefix to identify the entity and prevent any possible collisions across entity types.
For the RANGE value on the User entity, we’re using a static prefix of #METADATA# followed
by the username value. For the RANGE value, it’s important that you have a value that is
known, such as the username. This allows for single-item actions such as GetItem, PutItem,
and DeleteItem.
However, you also want a RANGE value with different values across different User entities to
enable even partitioning if you use this column as a HASH key for an index. For that reason,
you append the username to the RANGE key.
Second, the Photo entity is a child entity of a particular User entity. The main access pattern for
photos is to retrieve photos for a user ordered by date. Whenever you need something ordered
by a particular property, you will need to include that property in your RANGE key to allow for
sorting. For the Photo entity, use the same HASH key as the User entity, which will allow you to
retrieve both a user profile and the user’s photos in a single request. For the RANGE key, use
PHOTO#<USERNAME>#<TIMESTAMP> to uniquely identify a photo in your table.
Third, the Reaction entity is a child entity of a particular Photo entity. There is a one-to-many
relationship to the Photo entity and thus will use similar reasoning as with the Photo entity. In
the next module, you will see how to retrieve a photo and all of its reactions in a single query

using a secondary index. For now, note that the RANGE key for a Reaction entity is the same
pattern as the RANGE key for a Photo entity. For the HASH key, we use the username of the
user that is creating the reaction as well as the type of reaction applied. Appending the type of
reaction allows a user to add multiple reaction types to a single photo.
Finally, the Friendship entity uses the same HASH key as the User entity. This will allow you to
fetch both the metadata for a user plus all of the user’s followers in a single query. The RANGE
key for a Friendship entity is #FRIEND#<FRIEND_USERNAME>. In Step 4 below, you will learn
why to prepend the Friendship entity’s RANGE key with a “#”.
In the next step, we create a table with this primary key design 


## Create the DynamoDB Table

      # scripts/create_table.py
      import boto3
      
      dynamodb = boto3.client('dynamodb')
      
      try:
          dynamodb.create_table(
              TableName='quick-photos',
              AttributeDefinitions=[
                  {"AttributeName": "PK", "AttributeType": "S"},
                  {"AttributeName": "SK", "AttributeType": "S"}
              ],
              KeySchema=[
                  {"AttributeName": "PK", "KeyType": "HASH"},
                  {"AttributeName": "SK", "KeyType": "RANGE"}
              ],
              ProvisionedThroughput={
                  "ReadCapacityUnits": 10,
                  "WriteCapacityUnits": 10
              }
          )
          print("Table created successfully.")
      except Exception as e:
          print("Error creating table:", e)

   Run: 
        
         python3 scripts/create_table.py

<img width="530" height="81" alt="image" src="https://github.com/user-attachments/assets/800bc614-3d97-4032-a562-03d29c7c6698" />

check Dynamodb console to verify the table was created

<img width="959" height="239" alt="image" src="https://github.com/user-attachments/assets/69923421-6969-43dc-b820-e64f5a192d29" />


## Bulk Load Data

      # scripts/bulk_load_table.py
      import json
      import boto3
      
      dynamodb = boto3.resource('dynamodb')
      table = dynamodb.Table('quick-photos')
      
      items = []
      
      with open('scripts/items.json', 'r') as f:
          for row in f:
              items.append(json.loads(row))
      
      with table.batch_writer() as batch:
          for item in items:
              batch.put_item(Item=item)

Run:
         
         python3 scripts/bulk_load_table.py

To check if the DynamoDB table was loaded with items and get count, Run this command in your terminal: 
         
         aws dynamodb scan --table-name quick-photos --select "COUNT"

<img width="581" height="155" alt="image" src="https://github.com/user-attachments/assets/c2fecf19-74ef-4f9f-9612-be584c98599d" />



## Retrieve multiple entity types in a single request. The following code composes the fetch_user_and_photos.py script

      import boto3
      
      from entities import User, Photo
      
      dynamodb = boto3.client('dynamodb')
      
      USER = "jacksonjason"
      
      
      def fetch_user_and_photos(username):
          resp = dynamodb.query(
              TableName='quick-photos',
              KeyConditionExpression="PK = :pk AND SK BETWEEN :metadata AND :photos",
              ExpressionAttributeValues={
                  ":pk": { "S": "USER#{}".format(username) },
                  ":metadata": { "S": "#METADATA#{}".format(username) },
                  ":photos": { "S": "PHOTO$" },
              },
              ScanIndexForward=True
          )
      
          user = User(resp['Items'][0])
          user.photos = [Photo(item) for item in resp['Items'][1:]]
      
          return user
      
      
      user = fetch_user_and_photos(USER)
      
      print(user)
      for photo in user.photos:
          print(photo)


Run:
         
         python3 application/fetch_user_and_photos.py 


<img width="643" height="407" alt="image" src="https://github.com/user-attachments/assets/8b7c70e8-535a-439c-88bb-373e5fe6b0a4" />


 ## Create a secondary index
Creating a secondary index is similar to creating a table. Open the  add_inverted_index.py.
Inverted Index (GSI): Enables reverse lookups for relationships (e.g., users followed by a given user).

The contents of that file are shown below.
     
      import boto3
      
      dynamodb = boto3.client('dynamodb')
      
      try:
          dynamodb.update_table(
              TableName='quick-photos',
              AttributeDefinitions=[
                  {
                      "AttributeName": "PK",
                      "AttributeType": "S"
                  },
                  {
                      "AttributeName": "SK",
                      "AttributeType": "S"
                  }
              ],
              GlobalSecondaryIndexUpdates=[
                  {
                      "Create": {
                          "IndexName": "InvertedIndex",
                          "KeySchema": [
                              {
                                  "AttributeName": "SK",
                                  "KeyType": "HASH"
                              },
                              {
                                  "AttributeName": "PK",
                                  "KeyType": "RANGE"
                              }
                          ],
                          "Projection": {
                              "ProjectionType": "ALL"
                          },
                          "ProvisionedThroughput": {
                              "ReadCapacityUnits": 10,
                              "WriteCapacityUnits": 10 
                          }
                      }
                  }
              ],
          )
          print("Table updated successfully.")
      except Exception as e:
          print("Could not update table. Error:")
          print(e)


Whenever attributes are used in a primary key for the table or secondary index, they
must be defined in AttributeDefinitions. Then, we Create a new secondary index in the
GlobalSecondaryIndexUpdates property. For this secondary index, we specify the index
name, the schema of the primary key, the provisioned throughput, and the attributes we
want to project.
Note that an inverted index is a name of a design pattern rather than an official property
in DynamoDB. Creating an inverted index is just like creating any other secondary
index. 

Create your inverted index by running the command below.

      cd /home/ec2-user/environment/scripts
      python3 add_inverted_index.py


<img width="769" height="101" alt="image" src="https://github.com/user-attachments/assets/7affff57-8f0c-4350-9d49-65e30c8fa62d" />

<img width="959" height="289" alt="image" src="https://github.com/user-attachments/assets/e58e838b-75bc-4887-9352-ffcad739d284" />


## Query the inverted index to find a photo’s reactions 

Open the file: fetch_photo_and_reactions.py. The contents of this script are shown below.
      
      import boto3
      
      from entities import Photo, Reaction
      
      dynamodb = boto3.client('dynamodb')
      
      USER = "david25"
      TIMESTAMP = '2019-03-02T09:11:30'
      
      
      def fetch_photo_and_reactions(username, timestamp):
          try:
              resp = dynamodb.query(
                  TableName='quick-photos',
                  IndexName='InvertedIndex',
                  KeyConditionExpression="SK = :sk AND PK BETWEEN :reactions AND :user",
                  ExpressionAttributeValues={
                      ":sk": { "S": "PHOTO#{}#{}".format(username, timestamp) },
                      ":user": { "S": "USER$" },
                      ":reactions": { "S": "REACTION#" },
                  },
                  ScanIndexForward=True
              )
          except Exception as e:
              print("Index is still backfilling. Please try again in a moment.")
              return False
      
          items = resp['Items']
          items.reverse()
      
          photo = Photo(items[0])
          photo.reactions = [Reaction(item) for item in items[1:]]
      
          return photo
      
      
      photo = fetch_photo_and_reactions(USER, TIMESTAMP)
      
      if photo:
          print(photo)
          for reaction in photo.reactions:
              print(reaction)


The fetch_photo_and_reactions function is similar to a function you would have
in your application. The function accepts a username and timestamp and makes
a query against the InvertedIndex to find the photo and reactions for the photo.
Then it assembles the returned items into a Photo entity and multiple Reaction
entities that can be used in your application.

-  Run this command

        python3 fetch_photo_and_reactions.py

  Output:
  <img width="671" height="336" alt="image" src="https://github.com/user-attachments/assets/b68f79df-9143-431a-9e18-f3e8df3f7b5f" />


Find followed users
- In the previous step, you saw how to use an inverted index to fetch a one-to-
many relationship for an entity that was itself the subject of a one-to-many
relationship. In this step, you will use the inverted index to fetch the “other” side
of a many-to-many relationship.
-  The primary key in the table is allows you to find all of the followers of a particular user, but it won’t let you find all the users that someone is following. With the inverted index, it’s flipped -- you can find all the users followed by a particular
user.
  Open the file:

         find_following_for_user.py. The contents of this script follows.

---
      import boto3
      
      from entities import Friendship
      
      dynamodb = boto3.client('dynamodb')
      
      USERNAME = "haroldwatkins"
      
      
      def find_following_for_user(username):
          resp = dynamodb.query(
              TableName='quick-photos',
              IndexName='InvertedIndex',
              KeyConditionExpression="SK = :sk",
              ExpressionAttributeValues={
                  ":sk": { "S": "#FRIEND#{}".format(username) }
              },
              ScanIndexForward=True
          )
      
          return [Friendship(item) for item in resp['Items']]
      
      
      
      follows = find_following_for_user(USERNAME)
      
      print("Users followed by {}:".format(USERNAME))
      for follow in follows:
          print(follow)
      
---


Run the script by running the following command in your terminal.
      
      python3 find_following_for_user.py

Output:
<img width="769" height="158" alt="image" src="https://github.com/user-attachments/assets/ba2be4bb-fc4e-4c7e-8a3d-54327cbc391c" />

## Use partial normalization to find followed users
-Open the file: 
      
      find_and_enrich_following_for_user.py. 
      
The contents of this script are shown below 

      import boto3
      
      from entities import User
      
      dynamodb = boto3.client("dynamodb")
      
      USERNAME = "haroldwatkins"
      
      
      def find_and_enrich_following_for_user(username):
          friend_value = "#FRIEND#{}".format(username)
          resp = dynamodb.query(
              TableName="quick-photos",
              IndexName="InvertedIndex",
              KeyConditionExpression="SK = :sk",
              ExpressionAttributeValues={":sk": {"S": friend_value}},
              ScanIndexForward=True,
          )
      
          keys = [
              {
                  "PK": {"S": "USER#{}".format(item["followedUser"]["S"])},
                  "SK": {"S": "#METADATA#{}".format(item["followedUser"]["S"])},
              }
              for item in resp["Items"]
          ]
      
          friends = dynamodb.batch_get_item(RequestItems={"quick-photos": {"Keys": keys}})
      
          enriched_friends = [User(item) for item in friends["Responses"]["quick-photos"]]
      
          return enriched_friends
      
      
      follows = find_and_enrich_following_for_user(USERNAME)
      
      print("Users followed by {}:".format(USERNAME))
      for follow in follows:
          print(follow)


The find_and_enrich_following_for_user function is similar to the
find_follower_for_user function you used in the last module. The function accepts
a username for whom you want to find the followed users. The function first
makes a Query request using the inverted index to find all of the users that the
given username is following. It then assembles a BatchGetItem to fetch the full
User entity for each of the followed users and returns those entities.

-  This results in two requests to DynamoDB, rather than the ideal of one. However,
it’s satisfying a fairly complex access pattern, and it avoids the need to constantly
update Friendship entities every time a user profile is updated. This partial
normalization can be a great tool for your modeling needs.
-  Execute the script by running the following command in your terminal.

         ○ python3 find_and_enrich_following_for_user.py
Output:
<img width="901" height="377" alt="image" src="https://github.com/user-attachments/assets/13f065dc-d306-449e-99de-86ac2c35eabd" />

## React to a photo

-  When adding a user’s reaction to a photo, we need to do a few things:
-  Confirm that the user has not already used this reaction type on this photo
-  Create a new Reaction entity to store the reaction
-  Increment the proper reaction type in the reactions property on the Photo
entity so that we can display the reaction details on a photo

-  In the code you downloaded, there is a script in the application/ directory called
add_reaction.py that includes a function for adding a reaction to a photo. The
function in that file uses a DynamoDB transaction to add a reaction.
- The contents of the file are as follows:
      
      import datetime

      import boto3
      
      dynamodb = boto3.client("dynamodb")
      
      REACTING_USER = "kennedyheather"
      REACTION_TYPE = "sunglasses"
      PHOTO_USER = "ppierce"
      PHOTO_TIMESTAMP = "2019-04-14T08:09:34"
      
      
      def add_reaction_to_photo(reacting_user, reaction_type, photo_user, photo_timestamp):
          reaction = "REACTION#{}#{}".format(reacting_user, reaction_type)
          photo = "PHOTO#{}#{}".format(photo_user, photo_timestamp)
          user = "USER#{}".format(photo_user)
          try:
              resp = dynamodb.transact_write_items(
                  TransactItems=[
                      {
                          "Put": {
                              "TableName": "quick-photos",
                              "Item": {
                                  "PK": {"S": reaction},
                                  "SK": {"S": photo},
                                  "reactingUser": {"S": reacting_user},
                                  "reactionType": {"S": reaction_type},
                                  "photo": {"S": photo},
                                  "timestamp": {"S": datetime.datetime.now().isoformat()},
                              },
                              "ConditionExpression": "attribute_not_exists(SK)",
                              "ReturnValuesOnConditionCheckFailure": "ALL_OLD",
                          }
                      },
                      {
                          "Update": {
                              "TableName": "quick-photos",
                              "Key": {"PK": {"S": user}, "SK": {"S": photo}},
                              "UpdateExpression": "SET reactions.#t = reactions.#t + :i",
                              "ExpressionAttributeNames": {"#t": reaction_type},
                              "ExpressionAttributeValues": {":i": {"N": "1"}},
                              "ReturnValuesOnConditionCheckFailure": "ALL_OLD",
                          }
                      },
                  ]
              )
              print("Added {} reaction from {}".format(reaction_type, reacting_user))
              return True
          except Exception as e:
              print("Could not add reaction to photo")
      
      
      add_reaction_to_photo(REACTING_USER, REACTION_TYPE, PHOTO_USER, PHOTO_TIMESTAMP)


In the add_reaction_to_photo function, we’re using the transact_write_items()
method to perform a write transaction. Our transaction has two operations.

- First, we’re doing a Put operation to insert a new Reaction entity. As part of that
operation, we’re specifying a condition that the SK attribute should not exist for
this item. This is a way to ensure that an item with this PK and SK doesn’t
already exist. If it did, that would mean the user has already added this reaction
to this photo.

-  The second operation is an Update operation on the User entity to increment the
reaction type in the reactions attribute map. 

DynamoDB’s powerful update expressions allow you to perform atomic increments without needing to first retrieve the item and then update it.
- Run this script with the following command in your terminal.

      python3 add_reaction.py
  
- The output in your terminal should indicate that the reaction was added to the
photo.
-  Added sunglasses reaction from kennedyheather

<img width="561" height="293" alt="image" src="https://github.com/user-attachments/assets/a7eb441a-76b9-4df1-9db3-0ddf62a805a5" />

## Following a user
 In your application, one user can follow another user. When the application
backend gets a request to follow a user, we need to do four things:

- Check that the following user is not already following the requested user
- Create a Friendship entity to record the following relationship
- Increment the follower count for the user being followed
-  Increment the following count for the user following
-  Open the file:

         follow_user.py. 

The contents of the file are as follows:

      import datetime
      
      import boto3
      
      dynamodb = boto3.client('dynamodb')
      
      FOLLOWED_USER = 'tmartinez'
      FOLLOWING_USER = 'john42'
      
      
      def follow_user(followed_user, following_user):
          user = "USER#{}".format(followed_user)
          friend = "#FRIEND#{}".format(following_user)
          user_metadata = "#METADATA#{}".format(followed_user)
          friend_user = "USER#{}".format(following_user)
          friend_metadata = "#METADATA#{}".format(following_user)
          try:
              resp = dynamodb.transact_write_items(
                  TransactItems=[
                      {
                          "Put": {
                              "TableName": "quick-photos",
                              "Item": {
                                  "PK": {"S": user},
                                  "SK": {"S": friend},
                                  "followedUser": {"S": followed_user},
                                  "followingUser": {"S": following_user},
                                  "timestamp": {"S": datetime.datetime.now().isoformat()},
                              },
                              "ConditionExpression": "attribute_not_exists(SK)",
                              "ReturnValuesOnConditionCheckFailure": "ALL_OLD",
                          }
                      },
                      {
                          "Update": {
                              "TableName": "quick-photos",
                              "Key": {"PK": {"S": user}, "SK": {"S": user_metadata}},
                              "UpdateExpression": "SET followers = followers + :i",
                              "ExpressionAttributeValues": {":i": {"N": "1"}},
                              "ReturnValuesOnConditionCheckFailure": "ALL_OLD",
                          }
                      },
                      {
                          "Update": {
                              "TableName": "quick-photos",
                              "Key": {"PK": {"S": friend_user}, "SK": {"S": friend_metadata}},
                              "UpdateExpression": "SET following = following + :i",
                              "ExpressionAttributeValues": {":i": {"N": "1"}},
                              "ReturnValuesOnConditionCheckFailure": "ALL_OLD",
                          }
                      },
                  ]
              )
              print("User {} is now following user {}".format(following_user, followed_user))
              return True
          except Exception as e:
              print(e)
              print("Could not add follow relationship")
      
      follow_user(FOLLOWED_USER, FOLLOWING_USER)


 The follow_user function in the file is similar to a function you would have in your
application. It takes two usernames -- one of the followed user and one of the
following user -- and it runs a request to create a Friendship entity and update
the two User entities.
- Run the script in your terminal with the following command.

      python3 follow_user.py

-  You should see output in your terminal indicating that the operation succeeded.
-  User john42 is now following user tmartinez

  <img width="557" height="305" alt="image" src="https://github.com/user-attachments/assets/78607a2c-3952-41a2-a5ae-a236a10b7cf3" />


  ## In the previous sections, we’ve satisfied the following access patterns in our
application:
- Create user profile (Write)
- Update user profile (Write)
- Get user profile (Read)
- Upload photo (Write)
- View recent photos for user (Read)
- React to a photo (Write)
- View photo and reactions (Read)
- Follow user (Write)
- View followers for user (Read)
- View followed for user (Read)

## The strategies we used to satisfy these patterns included:
- A single-table design that combined multiple entity types in one table.
- A composite primary key that allow for many-to-many relationships.
- An inverted index to allow reverse lookups on our many-to-many entity.
- Partial normalization to keep our data fresh while remaining performant.\
- DynamoDB transactions to handle complex write patterns across multiple
items.

 ## Clean Up
 Delete the DynamoDB table: As part of the cleanup process, you need to delete the DynamoDB table you
used for this lab.
In the code you downloaded, there is a file in the scripts/ directory called
delete_table.py. The contents of that file are as follows.

       import boto3
       dynamodb = boto3.client('dynamodb')
      try:
      dynamodb.delete_table(TableName='quick-photos')
      print("Table deleted successfully.")
          except Exception as e:
             print("Could not delete table. Please try again in a moment. Error:")
      print(e)

Run:
   
    python3 delete_table.py

<img width="627" height="82" alt="image" src="https://github.com/user-attachments/assets/a683a91a-8fa8-4905-b987-00ec3e0c9a00" />

 ## Delete the AWS Cloud9 environment
- Navigate to the AWS Cloud9 console
- Choose the DynamoDB Quick Photos environment and choose Delete
- In the dialog box, type Delete in the box and choose Delete

Congratulations, you completed this project


The project successfully:

- Created and managed DynamoDB tables and indexes.

- Queried and updated data using boto3.

- Demonstrated transactional and batch operations.

- Implemented efficient single-table design.

## Conclusion

This project demonstrated the full lifecycle of designing, implementing, and managing a NoSQL data model for a social networking mobile application using Amazon DynamoDB.

---
## Key Takeaways

- Scalable single-table design with composite keys
- Use of secondary indexes for flexible queries
- Transactional writes ensuring consistency
- Practical use of partial normalization
- This design can scale for real-world applications like Instagram-style apps or photo-sharing platforms, providing both flexibility and speed.

---

