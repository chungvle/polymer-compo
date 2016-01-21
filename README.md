# PolymerCompo - A Polymer Composition Framework

PolymerCompo is a framework to support declarative composition of applications from components. 
In addition, it supports history management using pushState/popState.

PolymerCompo is implemented using Polymer 1.0. See the section on 'What is PolymerCompo' for more details.


## Demos
There are three demos in the demo directory:
  - flickr: searches flickr for pictures that contains a tag, e.g. a city name, entered by the user.
    This is a composition of four components: flickr-service, flickr-card, flickr-list, and input-header.
  - youtube: searches youtube for videos that contains a tag, e.g. a city name, entered by the user.
    This is a composition of four components: youtube-service, youtube-player, youtube-list, and input-header.
  - cityinfo: uses flickr and youtube to show pictures and vidoes. This is a composition of the above compositions.
  
To run the demos, follow the steps below:
  1. install polyserve
     npm install -g polyserve
  2. down load and unzip the framework into a directory
  3. cd to the directory in #2, then run bower install
  4. add your API keys:
    4.1. flickr: edit flickr-service.html in the folder elements to add 
    your flickr API key to the variable flickrKey
    4.2. youtube: edit youtube-service.html in the folder elements to add
    your youtube API key to the variable youtubeAPIKey
  5. next, run polyserve
  6. finally, open a browser and type in the following url:
     - for the flickr demo: http://localhost:8080/components/PolymerCompo/demo/flickr.html
     - for the youtube demo: http://localhost:8080/components/PolymerCompo/demo/youtube.html
     - for the cityinfo demo: http://localhost:8080/components/PolymerCompo/demo/index.html

## What is PolymerCompo?

PolymerCompo is a framework that supports composition of new applications from components. 
It consists of the following elements:
  - State: defines the interface to a component. 
  - Event: listens to changes of a target, e.g. outputs of a state, then broadcasts an event
    with the changed values.
  - Transition: listens to changes from an event, and processes a transition to the specified state.
  - State Manager: manages a collections of states, events, and transitions.
  - Composition: the simplest composition includes a state manager. In other cases, a composition might
    contain other compositions. Composition also defines an interface to start the application, 
    and to handle history management.
  - Relay: a relay is used to connect an event to another. 

To compose a new application using PolymerCompo:
  1. define an application shell: this is the main body of the application; it defines the
     containers for the views which are provided by the components.
  2. define a composition which includes the states, events and transitions. These elements capture
     the events that leads to transitions to new states which render new views. These views are placed
     in the corresponding containers defined in step #1.

One of our demos, flickr, showcases an application presenting Flickr photos for a city. 
The application shell and composition are described below.

The application shell: is defined in the element city-search-manager.html. It includes:
  - inputContainer: to contain the inputView; 
    the user enters a city name in this view as a tag to search for photos
  - listContainer: to contain the listView;
    the user clicks on a list item in this view to view a photo
  - detailContainer: to contain the detailView which displays the selected photo

The composition: the states, events, and transitions are defined as part of 
a state manager inside a composition in the element flickr-composition.html:
>    `<sc-composition>`
>        `<sc-manager>`
>            `<!-- states, events, transitions: see 'The Details' below --> `
>        `</sc-manager>`
>    `</sc-composition>`

Starting the composition: The application shell also contain a function to start the composition:
  attached: function() {
    this.$.cityinfoComposition.startApplication();
  }

The Details: The views in our composition are renderedby the following components:
  1. input-header.html: prompts the user to enter a tag to search for photos
  2. flickr-list.html: receives an input param, e.g. from input-header, and searches Flickr for photos
     that contains the keyword (or tag). Behind the scene, it uses flickr-service.html to query 
     Flickr for meta data.
  3. flickr-card.html: receives an input param, e.g. from an item selected by the user in
     flickr-list, and displays the corresponding photo.

The States: The above components are encapsulated by three corresponding states defined as follow:
  <sc-state id="flickrInputState" is-initial="true"
    component="input-header"
    params='["label", "title", "term"]' views='["inputView"]' outputs='["term"]'></sc-state>
  <sc-state id="flickrListState"
    component="flickr-list"
    params='["input"]' views='["listView"]' outputs='["selectedItem"]'></sc-state>
  <sc-state id="flickrDetailState"
    component="flickr-card"
    params='["item"]' views='["detailView"]'></sc-state>
    
params, views and outputs defines the name of the input param, view and output properties, respectively.

The Events: There are three events in our demo:
  sc-event id="flickrStartEvent"
    target='{"targetType": "application", "eventType": "applicationStart"}'
    event-values='[{"name": "label", "value": "Enter City Name"},
             {"name": "title", "value": "City Search"}]'
  This event sends the signal to start the application.
  
  <sc-event id="flickrCitySelectedEvent"
    target='{"targetType": "state", "id": "flickrInputState"}'></sc-event>
  This event sends the city name entered as a tag in flickrInputState.
  
  <sc-event id="flickrItemSelectedEvent"
    target='{"targetType": "state", "id": "flickrListState"}'></sc-event>
  This event sends the list item selected by the user in flickrListState.
  
There is also a special event when the term property of flickrInputState is an empty string.
This event is used to reset the states.
  <sc-event id="flickrResetEvent"
    target='{"targetType": "state", "id": "flickrInputState", "eventType": "stateReset"}'
    event-values='[{"name": "term", "value": ""}]'></sc-event> 
    
The Transitions: The following transitions are defined to respond to the above events:
  <sc-transition id="flickrInputTransition" 
    events='["flickrStartEvent"]' transition-to="flickrInputState" 
    containers='[{"inputView": "inputContainer"}]'></sc-transition> 
  It responds to flickrStartEvent, sets the event-values to the input params of flickrInputState,
  and assigns inputView defined in flickrInputState to inputContainer.
  
  <sc-transition id="flickrListTransition"
    events='["flickrCitySelectedEvent", "flickrResetEvent"]' transition-to="flickrListState" 
    containers='[{"listView": "flickrListContainer"}]'></sc-transition>
  It responds to flickrCitySelectedEvent, sets the event-values to the input params of flickrListState,
  and assigns listView defined in flickrListState to flickrListContainer. Notice that this transition
  also responds to flickrResetEvent by removing listView.
  
  <sc-transition id="flickrDetailTransition"
    events='["flickrItemSelectedEvent", "flickrResetEvent"]' transition-to="flickrDetailState" 
    containers='[{"detailView": "detailContainer"}]'></sc-transition>
  It responds to flicktDetailTransition, sets the event-values to the input params of flickrDetailState,
  and assign detailView defined in flickrDetailState to detailContainer. This transition also
  responds to flickrResetEvent by removing detailView.

History Management: Navigation from state to state is captured in the url of the application by using
pushState, or replaceState if the transition involves the same containers. The application also
listens to the popState event, e.g. when the user clicks on the back or forward button. When that happens,
PolymerCompo uses the history object stored with pushState/replaceState to reload the current page.
History management is implemented by the element sc-history.
