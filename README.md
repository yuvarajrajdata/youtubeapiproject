# Project Title :
      YouTube Data Harvesting and Warehousing using SQL, MongoDB and Streamlit
     
# Skills take away From This Project :
      Python scripting, Data Collection,MongoDB, Streamlit, API integration, Data Managment using MongoDB (Atlas) and SQL.......


import streamlit as st
from googleapiclient.discovery import build
import pymongo
import pymysql
import mysql.connector



import streamlit as st
from streamlit_bokeh_events import streamlit_bokeh_events


custom_css = """
<style>
body {
    background-color: #f5f5f5;
    font-family: Arial, sans-serif;
}
h1 {
    color: #FF0000;
    font-size: 40px;
    font-weight: bold;
    margin-bottom: 16px;
}
</style>
"""
st.markdown(custom_css, unsafe_allow_html=True)

# Rest of your Streamlit app code
# ...


# YouTube API key
api_key = "AIzaSyCKSZepfYh4nv061aTkvsKRhyO4uxVJAOc"
youtube = build('youtube', 'v3', developerKey=api_key)

# Streamlit app layout
st.title("YouTube Channel Data")


# Connect to MongoDB database
client = pymongo.MongoClient("mongodb+srv://yuvarajspt1998:Raj19980@cluster123.itdlmho.mongodb.net/")
collection = client['youtubedata']['channeldetails']

# Retrieve and display channel data
def retrieve_channel_data(channel_id):
    try:
        response = youtube.channels().list(
            part='snippet,statistics',
            id=channel_id
        ).execute()

        if 'items' in response:
            item = response['items'][0]
            statistics = item['statistics']

            data = {
                'channel_name': item['snippet']['title'],
                'subscribers': statistics.get('subscriberCount', 0),
                'views': statistics.get('viewCount', 0),
                'total_videos': statistics.get('videoCount', 0),
                'likes': statistics.get('likeCount', 0),
                'dislikes': statistics.get('dislikeCount', 0),
                'playlist_id': '',
                'video_id': item['id']
            }

            if 'contentDetails' in item:
                data['playlist_id'] = item['contentDetails']['relatedPlaylists']['uploads']

            collection.insert_one(data)
            st.success("Data inserted into MongoDB.")

            # Display channel data
            st.subheader("Channel Name")
            st.write(data['channel_name'])
            st.subheader("Channel Statistics")
            st.write(f"Subscribers: {data['subscribers']}")
            st.write(f"Views: {data['views']}")
            st.write(f"Total Videos: {data['total_videos']}")
            st.write(f"Likes: {data['likes']}")
            st.write(f"Dislikes: {data['dislikes']}")
            
        else:
            st.error("Channel not found.")
    except Exception as e:
        st.error(f"An error occurred: {e}")

# Get YouTube Channel IDs from user input
channel_ids = st.text_input("Enter YouTube Channel IDs (comma-separated)").split(',')

# Retrieve and store channel data
if st.button("Get Data & migrate to mongodb"):
    for channel_id in channel_ids:
        retrieve_channel_data(channel_id.strip())

        



# Connect to SQL database
sql_connection = pymysql.connect(
    host="localhost",
    user="root",
    password="Raj@199807886",
    database="youtubedata"
)

# Retrieve available channel names from MongoDB
channel_names = [data['channel_name'] for data in collection.find({}, {'channel_name': 1})]

# Select a channel name
selected_channel = st.selectbox("Select a Channel Name", channel_names)

# Migrate data to SQL database
if st.button("Migrate Data to MySql"):
    try:
        # Retrieve data from MongoDB
        data = collection.find_one({'channel_name': selected_channel})      
        
              
        
        
        # Create SQL tables and insert data
        with sql_connection.cursor() as cursor:
            # Create a table for channel statistics
            create_table_query = """
            CREATE TABLE IF NOT EXISTS channel_statistics (
                channel_name VARCHAR(255),
                subscribers INT,
                views VARCHAR(255),
                total_videos INT,
                likes INT,
                dislikes INT,
                playlist_id VARCHAR(255),
                video_id VARCHAR(255)
            )
            """
            cursor.execute(create_table_query)

            # Insert data into the channel statistics table
            insert_data_query = """
            INSERT INTO channel_statistics
            (channel_name, subscribers,total_videos, likes, dislikes, playlist_id, video_id)
            VALUES (%s, %s, %s, %s, %s, %s, %s)
            """
            
                    
            
            cursor.execute(insert_data_query, (
                data['channel_name'], data['subscribers'],data['total_videos'],
                data['likes'], data['dislikes'], data['playlist_id'], data['video_id']
            ))

        sql_connection.commit()
        st.success("Data migration to SQL database is successful.")
    except Exception as e:
        st.error(f"Error migrating data: {e}")

# Close the MongoDB and SQL connections
client.close()
sql_connection.close()
        
        

import urllib.parse
import sqlalchemy


# URL-encode the password
password = urllib.parse.quote_plus("Raj@199807886")

# Create the connection string with the URL-encoded password
connection_string = f"mysql+pymysql://root:{password}@localhost/youtubedata"

# Create the SQLAlchemy engine
engine = sqlalchemy.create_engine(connection_string)

# Retrieve available channel names from the SQL database
with engine.connect() as connection:
    result = connection.execute("SELECT channel_name FROM channel_statistics")
    channel_names = [row[0] for row in result.fetchall()]

# Retrieve user input for the channel name
selected_channel = st.selectbox("Select a Channel Name", channel_names, key="channel_select")

# Execute an SQL query to retrieve data for the specified channel
query = f"SELECT * FROM channel_statistics WHERE channel_name = '{selected_channel}'"

# Execute the query and fetch the results
with engine.connect() as connection:
    result = connection.execute(query)
    data = result.fetchall()

# Display the retrieved data in a table
st.table(data)


# Example: Bar chart of subscribers and views
subscribers = [row[1] for row in data]
views = [row[2] for row in data]
chart_data = {"Subscribers": subscribers, "Views": views}
st.bar_chart(chart_data)








