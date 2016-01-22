# PolymerCompo - A Polymer Composition Framework

PolymerCompo is a framework to support declarative composition of applications from components. 
In addition, it supports history management using pushState/popState.

PolymerCompo is implemented using Polymer 1.0. See the section on 'What is PolymerCompo' for more details.


## Demos
There are three demos in the demo directory:
  - <b>flickr</b>: searches flickr for pictures that contains a tag, e.g. a city name, entered by the user.
    This is a composition of four components: flickr-service, flickr-card, flickr-list, and input-header.
  - <b>youtube</b>: searches youtube for videos that contains a tag, e.g. a city name, entered by the user.
    This is a composition of four components: youtube-service, youtube-player, youtube-list, and input-header.
  - <b>cityinfo</b>: uses flickr and youtube to show pictures and vidoes. This is a composition of the flickr and youtube compositions.
  
To run the demos, follow the steps below:
  1. install polyserve
     npm install -g polyserve
  2. down load and unzip the framework into a directory
  3. cd to the directory in #2, then run bower install
  4. add your API keys:
    - flickr: edit flickr-service.html in the folder elements to add 
    your flickr API key to the variable flickrKey. See [Flickr API] (https://www.flickr.com/services/api/misc.api_keys.html "Click for details on Flickr API") 
    for more detail.
    - youtube: edit youtube-service.html in the folder elements to add
    your youtube API key to the variable youtubeAPIKey. See [YouTube Data API] (https://developers.google.com/youtube/v3/getting-started "Click for details on YouTube Data API") 
    for more detail.
  5. next, run polyserve
  6. finally, open a browser and type in the following url:
     - for the flickr demo: http://localhost:8080/components/PolymerCompo/demo/flickr.html
     - for the youtube demo: http://localhost:8080/components/PolymerCompo/demo/youtube.html
     - for the cityinfo demo: http://localhost:8080/components/PolymerCompo/demo/index.html

## What is PolymerCompo?

PolymerCompo is a framework that supports composition of new applications from components. 
It consists of the following elements:
  - <b>State</b>: defines the interface to a component. 
  - <b>Event</b>: listens to changes of a target, e.g. outputs of a state, then broadcasts an event
    with the changed values.
  - <b>Transition</b>: listens to changes from an event, and processes a transition to the specified state.
  - <b>State Manager</b>: manages a collections of states, events, and transitions.
  - <b>Composition</b>: the simplest composition includes a state manager. In other cases, a composition might
    contain other compositions. Composition also defines an interface to start the application, 
    and to handle history management.
  - <b>Relay</b>: a relay is used to connect an event to another. 

To compose a new application using PolymerCompo:
  1. define an <b>application shell</b>: this is the main body of the application; it defines the
     containers for the views which are provided by the components.
  2. define a <b>composition</b> which includes the states, events and transitions. These elements capture
     the events that leads to transitions to new states which render new views. These views are placed
     in the corresponding containers defined in step #1.

One of our demos, flickr, showcases an application presenting Flickr photos for a city. 
The application shell and composition are described below.

<b>The application shell:</b> is defined in the element city-search-manager.html. It includes:
  - inputContainer: to contain the inputView; 
    the user enters a city name in this view as a tag to search for photos
  - listContainer: to contain the listView;
    the user clicks on a list item in this view to view a photo
  - detailContainer: to contain the detailView which displays the selected photo

<b>The composition:</b> the states, events, and transitions are defined as part of 
a state manager inside a composition in the element flickr-composition.html:
<!-- language: lang-html -->
    <sc-composition>
        <sc-manager>
            <!-- states, events, transitions: see 'The Details' below -->
        </sc-manager>
    </sc-composition>

<b>Starting the composition:</b> The application shell also contain a function to start the composition:
<!-- language: lang-js -->
    attached: function() {
        this.$.cityinfoComposition.startApplication();
    }

<b>The Details:</b> The views in our composition are renderedby the following components:
  1. input-header.html: prompts the user to enter a tag to search for photos
  2. flickr-list.html: receives an input param, e.g. from input-header, and searches Flickr for photos
     that contains the keyword (or tag). Behind the scene, it uses flickr-service.html to query 
     Flickr for meta data.
  3. flickr-card.html: receives an input param, e.g. from an item selected by the user in
     flickr-list, and displays the corresponding photo.

<b>The States:</b> The above components are encapsulated by three corresponding states defined as follow:
<!-- language: lang-html -->
    <sc-state id="flickrInputState" is-initial="true"
        component="input-header"
        params='["label", "title", "term"]' views='["inputView"]' outputs='["term"]'></sc-state>
    <sc-state id="flickrListState"
        component="flickr-list"
        params='["input"]' views='["listView"]' outputs='["selectedItem"]'></sc-state>
    <sc-state id="flickrDetailState"
        component="flickr-card"
        params='["item"]' views='["detailView"]'></sc-state>
    
params, views and outputs define the name of the input param, view and output properties, respectively.

<b>The Events:</b> There are three events in our demo:
<!-- language: lang-html -->
    <sc-event id="flickrStartEvent"
        target='{"targetType": "application", "eventType": "applicationStart"}'
        event-values='[{"name": "label", "value": "Enter City Name"},
            {"name": "title", "value": "City Search"}]'></sc-event>
The above event sends the signal to start the application.
  
<!-- language: lang-html -->
    <sc-event id="flickrCitySelectedEvent"
        target='{"targetType": "state", "id": "flickrInputState"}'></sc-event>
The above event sends the city name entered as a tag in flickrInputState.

<!-- language: lang-html -->  
    <sc-event id="flickrItemSelectedEvent"
        target='{"targetType": "state", "id": "flickrListState"}'></sc-event>
The above event sends the list item selected by the user in flickrListState.
  
There is also a special event when the term property of flickrInputState is an empty string.
It is used to reset the states.
<!-- language: lang-html -->
    <sc-event id="flickrResetEvent"
        target='{"targetType": "state", "id": "flickrInputState", "eventType": "stateReset"}'
        event-values='[{"name": "term", "value": ""}]'></sc-event> 
    
<b>The Transitions:</b> The following transitions are defined to respond to the above events:
<!-- language: lang-html -->
    <sc-transition id="flickrInputTransition" 
        events='["flickrStartEvent"]' transition-to="flickrInputState" 
        containers='[{"inputView": "inputContainer"}]'></sc-transition> 
It responds to flickrStartEvent, sets the event-values to the input params of flickrInputState,
and assigns inputView defined in flickrInputState to inputContainer.

<!-- language: lang-html -->
    <sc-transition id="flickrListTransition"
        events='["flickrCitySelectedEvent", "flickrResetEvent"]' transition-to="flickrListState" 
        containers='[{"listView": "flickrListContainer"}]'></sc-transition>
It responds to flickrCitySelectedEvent, sets the event-values to the input params of flickrListState,
and assigns listView defined in flickrListState to flickrListContainer. Notice that this transition
also responds to flickrResetEvent by removing listView.

<!-- language: lang-html -->
    <sc-transition id="flickrDetailTransition"
        events='["flickrItemSelectedEvent", "flickrResetEvent"]' transition-to="flickrDetailState" 
        containers='[{"detailView": "detailContainer"}]'></sc-transition>
It responds to flicktDetailTransition, sets the event-values to the input params of flickrDetailState,
and assign detailView defined in flickrDetailState to detailContainer. This transition also
responds to flickrResetEvent by removing detailView.

<b>History Management:</b> Navigation from state to state is captured in the url of the 
application by using pushState, or replaceState if the transition involves the same containers. 
The application also listens to the popState event, e.g. when the user clicks on the back 
or forward button. When that happens, PolymerCompo uses the history object stored with 
pushState/replaceState to reload the current page. 
History management is implemented by the element sc-history.
