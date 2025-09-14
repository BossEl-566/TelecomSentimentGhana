# Command to get comment data from Youtube and put it in data form
 

!pip install google-api-python-client pandas tqdm --quiet

from googleapiclient.discovery import build
import pandas as pd
from tqdm.notebook import tqdm

# === CONFIG ===
API_KEY = "AIzaSyA2Ii6g0gFZUb49BfTAgiWiScxtJ6WmOzI"  # paste your API key
VIDEO_IDS = ["a35pZ1rPXKM", "4B-OD2zKfb8","vP62n8VU1ns","JD6AobWSE2k","iBKx52sBO_g","mTIyBri1UVc","8-nQjzbhLfs","d--qNa3OGR4","ZNLe6NZZ9zA","NpuBWD5sWrA","_XUBnbTMxUI","N7GU3ffs9aU","LyE3KB36D9c","BUc7PtGd20k","IKFox3iurbM","fxG6ISdk8ao","1l-MxhWEABk","GfMhYFcWEjc","UPu0kZJpMGo","92DXLGKmob0","QyFzLu49UjY","tl_MK8yBVmc","SmqdRQq9MJc","Ef0wj2gFDEg","1KOIMJIU1s4", ]  # add your chosen video IDs

youtube = build('youtube', 'v3', developerKey=API_KEY)

def get_comments(video_id, max_results=100):
    comments = []
    request = youtube.commentThreads().list(
        part="snippet",
        videoId=video_id,
        maxResults=max_results,
        textFormat="plainText"
    )
    response = request.execute()

    while response:
        for item in response['items']:
            comment = item['snippet']['topLevelComment']['snippet']
            comments.append({
                'video_id': video_id,
                'author': comment['authorDisplayName'],
                'text': comment['textDisplay'],
                'published_at': comment['publishedAt'],
                'like_count': comment['likeCount']
            })
        if 'nextPageToken' in response:
            response = youtube.commentThreads().list(
                part="snippet",
                videoId=video_id,
                pageToken=response['nextPageToken'],
                maxResults=max_results,
                textFormat="plainText"
            ).execute()
        else:
            break
    return comments

all_comments = []
for vid in tqdm(VIDEO_IDS):
    all_comments.extend(get_comments(vid))

df = pd.DataFrame(all_comments)
df.to_csv('/content/telecom_comments.csv', index=False)
df.head()
