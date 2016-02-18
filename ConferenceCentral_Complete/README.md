# Conference Organization App

## About

Conference Organization App is a cloud-based api server which readily supports web-based applications but can easily be updated to support Android and iOS applications, as well. The Conference Organization API is hosted on Google App Engine while using the google ndb database. Endpoints are made available for various applications to access the data.

Functionality:
 - User Authentication via Google
 - Create/Edit/View/Delete User profiles
 - Create/Edit/View/Delete Conferences and conference information 
 - Create/Edit/View/Delete sessions for conferences (including a user wishlist)

Accessing the Application:
 - Application ID = ekmark-project
 - Application UI = https://ekmark-project.appspot.com
 - Application API Explorer = https://ekmark-project.appspot.com/_ah/api/explorer

## Sessions and Speaker Design

The Session model design located in models.py is as follows:

| Property             | Type             |
|----------------------|------------------|
| name                 | string, required |
| highlights           | string           |
| speaker              | string           |
| duration             | integer          |
| typeOfSession        | string           |
| date                 | date             |
| startTime            | time             |
| organizerUserId      | string           |
| month                | string           |
| sponsor              | string           |
| building             | string           |
| websafeConferenceKey | string           |
| websafeSessionKey    | string           |

The websafeConferenceKey is a copy of the urlsafe key of the parent conference for the created session. This was implemented to help keep track of which session belongs to which conference. I also copied the urlsafe key of the created session into the database for a session so that I could use easily find and use it when needed.

Endpoint Methods:

 - `createSession`: Create a session for a given conference.
 - `getConferenceSessions`: Get all sessions for a given conference.
 - `getConferenceSessionsByType`: Get all sessions for a given conference, but filtered by session type.
 - `getSessionsBySpeaker`: Get all sessions, for all conference, for a given speaker.

In order for a session to be connected to its conference, each session was given that conference as its parent via that conference's websafeconferenceKey. This key was also passed into the Session model for visual verification purposes. Due to the fact that a conference can have multiple sessions, placing a copy of the conference's websafeConferenceKey in it's session's data makes it easier to visually keep track of a conference's sessions. 

In terms of the speaker, there were a couple of options for implementation. A speaker model class could have been implemented, but this would have added another layer of code in terms of configuring the connection between the speaker and session data. Although this connection would not be difficult to establish, it would be much easier to establish the connection simply by creating a property within the session class called speaker. Calling a session's speaker as necessitated in Task 4 is do-able, but there are several limitations. For instance, if a speaker wished to upload/edit/delete information about himself/herself, this would be difficult to do, unless he/she created the session. If I had implimented a speaker class, permission could be given to a speaker to upload/edit his own information. This, however, could be seen as a limitation itself, since creating a session would then inherently require two people (the session creater and the speaker).

## Wishlist

Endpoints:

 - `addSessionToWishlist`: Add a session's key to the user's profile data (added a sessKey property to User table).
 - `getSessionsInWishlist`: Return a user's wishlist. This is achieved by getting the session keys in a user's profile.
 - 'deleteSessionsInWishlist': Delete a session in user's wishlist by removing that session key from user's profile.

## Indexes and Queries

Items were automatically indexed into the google datastore index by running Conference Organizer locally. There were no composit indexes necessary for session queries, thus, the index.yaml file is unchanged.

Two additional queries:

 - 'getSessionsByBuilding': This query was implemented in order to allow user's to find sessions which might be in the same building. Conferences often have sessions in various building. This query allows users to plan ahead and attend sessions in the same building.

 - 'getSessionsBySponsor': Many people look for a session's sponsor when choosing to attend that session. A sponsor should have a great foundation in the session type they are offering, therefore, one would look to see a session's sponsor in order to judge the validity of that session.

Handling a query for non-workshop sessions before 7:00 pm:

 - The problem with the query noted above is that there are inequality statements being placed upon two properties (which is invalid when querying the google database). One solution to this problem is pretty straight-forward. First, one would query the database for all sessions which are not workshops. One could then run a for loop through the objects obtained in the query and remove all of the sessions with a start time after 7:00 pm (sessions.remove(session)).

## Featured Speaker

I implemented a taskqueue when a session object is created which places the name of the session as well as the name of the speaker for that session in the memcache only if that speaker is also the speaker of a different session at the same conference. 

Endpoint:
 - 'getFeaturedSpeaker': gets the featured speaker from memcache and displays a string message stating that this speaker is also speaking at other sessions. 

## Setup Instructions

To deploy this API server locally, ensure that you have downloaded and installed the [Google App Engine SDK for Python](https://cloud.google.com/appengine/downloads). Once installed, conduct the following steps:

1. Clone this repository. Only the `p4` directory is essential to this project.
2. (Optional) Update the value of `application` in `app.yaml` to the app ID you have registered in the App Engine admin console and would like to use to host your instance of this sample.
3. (Optional) Update the values at the top of `settings.py` to reflect the respective client IDs you have registered in the [Developer Console][4].
4. (Optional) Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
5. (Optional) Mark the configuration files as unchanged as follows: `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
6. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
7. (Optional) Generate your client library(ies) with [the endpoints tool][6].
8. (Optional) Deploy the application via `appcfg.py update`