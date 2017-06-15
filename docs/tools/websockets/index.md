
# UNDER CONSTRUCTION!!!


# Websockets

## Introduction

This tool has been made available in order to provide real-time updates from assets in theOrganiCity platform. As an example, I might want to get a notification if the lights suddenly turns on in my office. By having e.g. a smartphone application that is using the websockets tool, it becomes possible to listen for updates coming directly from the office (provided that the office's light source is registerede as an asset in the OrganiCity platform). More generally, this tool creates a websocket client for connecting with the OrganiCity platform, and subscribe to updates on assets already registered on the platform.

## Installation

In order to make this tool work one would need to be working in a Java environment, and it is therefore a prerequisite that a java IDE is installed. Otherwise, the installation is quite simple.

1. Go to this Github page and download the project as a zip file: https://github.com/OrganicityEu/WebsocketClient
2. Unzip the file locally
3. You can either open the entire websockets project as a maven project in your favourite java IDE or simply import WebsocketsClient/libs/WebsocketClient.jar as an external jar into your existing project

That is it! In the following section, we will provide a concrete usage example on how to import the websocket tool as an externalt jar into a new java project.

## Usage example

This section provides a simple example on how to use the websocket tool, and we will be using Netbeans as the java IDE. The specific example shows how to subscribe to a single asset already created in the OrganiCity platform (it is not possible to subscribe to more than one asset at a time, and it is not possible to subscribe to creation or deletion of a new asset).

### Start Netbeans and create an new empty java project

![TSmarT OC configuration main](images/OC_TSmarT_web_01.JPG)

!(images/OC_TSmarT_web_01.JPG)


```
import java.util.List;
import dk.alexandra.orion.websocket.transports.OrionSubscription;
import websockets.WebsocketClientStomp;
import websockets.handlers.WebsocketCallback;

public class VanillaTestClient implements WebsocketCallback{
  
  WebsocketClientStomp wsc;
  long now;
  boolean hasSubscription = false;
  
  
  public VanillaTestClient(){
    now = System.currentTimeMillis();
    wsc= new WebsocketClientStomp(this, "ws://31.200.243.76:8090/orion");
    startLoop();
  }
  
  private void startLoop(){
    while(true){
      if(System.currentTimeMillis()>now+5000){
        if(!hasSubscription){
			//example of how to create a subscription
			String[] attr = new String[1];
			attr[0] = "comments";
			String[] cond = new String[1];
			cond[0] = "comments";
			String entityId = "urn:oc:entity:experimenters:5a660d96-0ef7-42ca-9f6c-5dbb86d6aa20:58ab32f36f8b513746565c54:wsasset";

			String token = "<Your token here>";
			OrionSubscription subscription = new OrionSubscription(cond, attr, "P1D", entityId, false, "urn:oc:entityType:userImage",null,token);
					  
			System.out.println("trying to set subscrition");
			hasSubscription = wsc.registerSubscription(subscription);

            if(hasSubscription){
              System.out.println("subscription set");
            }  
          }else{
            List<String> subscriptions = wsc.getSubscriptions();
            if(subscriptions.size()>0){
              if(wsc.unregisterSubscription(subscriptions.get(0))){
                hasSubscription=false;
                
              }
            }
          }
        
        now = System.currentTimeMillis();
      }
    }
  }
  

  @Override
  public void messageReceived(String mesg) {
    System.out.println("Received message in Processing "+mesg);
  }
  
  
  
  public static void main(String[] args){
    System.out.println("Starting Vanilla test client");
    VanillaTestClient client = new VanillaTestClient();
  }

}
```


## Technical backrgound

The OrganiCity platform is built around the Orion Context Broker (https://fiware-orion.readthedocs.io/en/master/), and the websockets tool conform to the abilities and formats dictated by the context broker. As the Context Broker is REST based, we provide a Spring.io based middleware (https://github.com/OrganicityEu-Platform/WebsocketMiddleware) for transforming the connections between REST and Websockets.

For further technical and code detail, please take a look here: https://github.com/OrganicityEu/WebsocketClient

reference: https://fiware-orion.readthedocs.io/en/develop/user/walkthrough_apiv2/index.html#subscriptions

Look there to acquire token: https://github.com/OrganicityEu/organicityeu.github.io/blob/mkdocs/docs/HowToAuthenticateAnUser.md