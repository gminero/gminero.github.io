---
title: "Lwc Aura Wrappers"
date: 2019-04-08T10:52:13-04:00
draft: true
---

In this post, we will create an Aura wrapper component around or LWC that fires an APP event to an external Aura component.
Wrapping your LWC will help you communicate with other aura components that cannot or have not been migrated to LWC.
## Getting Started

You will need to get yourself setup with [Salesforce-DX](https://trailhead.salesforce.com/en/content/learn/trails/sfdx_get_started), if you haven't had the chance to do so, this is a good time to get your environemnt setup since we will use it to deploy our LWC to a scratch org for testing.

## Steps To do 

- [Start your project](#Start-your-project)
- [Create your LWC](#create-your-lwc)
- [Component Markup](#component-markup)
- [Component Controller](#component-controller)
- [Create your Aura Wrapper](#create-your-aura-wrapper)
- [Aura Component Markup](#aura-component-markup)
- [Aura Component Controller](#aura-component-controller)
- [Create your Aura Listener](#create-your-aura-listener)
- [Listener Component Markup](#listener-component-markup)
- [Listener Component Controller](#listener-component-controller)

---

## Start your project

> sample project-scratch-def.json

```

{
  "orgName": "CaseCreation Demo",
  "edition": "Developer",
  "features": ["Communities", "Sites", "SiteDotCom"],
  "hasSampleData": "false",
  "settings": {
      "orgPreferenceSettings": {
          "s1DesktopEnabled": true,
          "selfSetPasswordInApi": true,
          "s1EncryptedStoragePref2": false,
          "networksEnabled": true
      }
  }
} 
```

---
run 

```
sfdx force:org:create -f project-scratch-def.json -a myCaseCreationTest
```

You can either create a lightinng community or push the data from your scratch org if you already have one setup in a scratch org. If you are unfamilair with the process, you can refer to [Create a Community](https://trailhead.salesforce.com/en/content/learn/projects/communities_theme_layout/create_community_theme).

I have additionaly setup the following github repository if you would like to push a sample community and load test data:
[!Joe Coffee Repository](https://github.com/gminero/joe-coffee)

---

## Create your LWC

> [lightning commads](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_force_lightning.htm)

the following command will create a lightning web component in the appropriate directory:

```
sfdx force:lightning:component:create -n lwcContactSupport --type lwc -d force-app/main/default/lwc
```

{{< figure src="/img/lwcLayoutForm.png" title="layout form" >}}

---
## Component Markup
We will be using  standard [lighting-input](https://developer.salesforce.com/docs/component-library/bundle/lightning-input/example)
 & [lightning-textarea](https://developer.salesforce.com/docs/component-library/bundle/lightning-textarea/example) components for the form.
 
 Additionally, we will leverage [lightning-layout](https://developer.salesforce.com/docs/component-library/bundle/lightning-layout/example) components
 for the layout.


```
<template>
    
    <lightning-layout multiple-rows="true">
        <lightning-layout-item padding="around-small">
            <div class="slds-text-heading_medium slds-m-bottom_medium">
                Sample LWC record edit form
            </div>
        </lightning-layout-item>
        <lightning-layout-item padding="around-small" size="12">
            <lightning-input class='required' label="Subject"
                name="subject" onchange={logSomething}> </lightning-input>
        </lightning-layout-item>
        <lightning-layout-item padding="around-small" size="12">
            <lightning-textarea class='required' label="Description"
                name="description" onchange={logSomething}> </lightning-textarea>
        </lightning-layout-item>
                
        <lightning-layout-item padding="around-small">
                <button type="button" name="Submit Case" value="Submit Case" class="stretchButton slds-button slds-button_brand" onclick={handleClick} >Submit Case</button>
            </lightning-layout-item>
                
    </lightning-layout>
        
</template>


```
				
| Important: in this component, the button does nothing! |
| --- |

---

## Component Controller

> [Create and Dispatch Events](https://developer.salesforce.com/docs/component-library/documentation/lwc/events_create_dispatch)

In the controller, we create and dispatch a custom event for the Aura component to handle.

```
		
import { LightningElement, track } from 'lwc';

let queryContext = {};

export default class ContactSupportForm extends LightningElement {
    @track clickedButtonLabel;
    
    handleClick(event) {
        this.clickedButtonLabel = event.target.name;
    }

    logSomething(event){

        const fieldChangeEvent = new CustomEvent('fieldchange', {
           detail : {'fieldName': event.target.name,
                    'fieldValue' : event.target.value}
        });
        // Fire the custom event
        this.dispatchEvent(fieldChangeEvent);
    }
}

```
		

---

## Create your Aura Wrapper

We will create a lightning aura component that will wrap our lwc. The only purpose of the aura wrapper component is to fire application
events for our other Aura compontent(s) to handle.


## Aura Component Markup

`selfService:caseCreateFieldChange` is a non-documented event, you can use it to pass field/value attribute names in between components
in the context of the contact-support page in lightning communities.

```
<!-- auraDomEventListener.cmp -->
<aura:component implements="forceCommunity:availableForAllPageTypes">
    <aura:attribute name="timer" type="Integer"/>
    <aura:registerEvent name="appEvent" type="selfService:caseCreateFieldChange"/>  
    
    <lightning:card title="AuraDomEventListener" iconName="custom:custom30">
        <aura:set attribute="actions">
            <span class="aura">Aura Component</span>
        </aura:set>
        <div class="slds-m-around_medium slds-align_absolute-center">
            <lightning:layout>
                <lightning:layoutItem size="12" class="slds-box">
                    
                    <!-- This is the LWC component -->
                    <c:contactSupportForm onfieldchange="{!c.handleFieldChange}"/>

                </lightning:layoutItem>
            </lightning:layout>
        </div>
    </lightning:card>
</aura:component>


```
				
| Important: lwc events prepend an `on` to the name, this means that `fieldchange` becomes `onfieldchange` ! |
| --- |

---

## Aura Component Controller

> [Application Events](https://developer.salesforce.com/docs/atlas.en-us.lightning.meta/lightning/events_application_example.htm)

In the controller, we fire the `caseCreateFieldChange` selfService event for the Aura Listener component to handle.

```
		
({
    handleFieldChange : function(cmp, event, helper) {       

        var appevt = $A.get("e.selfService:caseCreateFieldChange");
        appevt.setParams({
            "modifiedField": event.getParam('fieldName'),
            "modifiedFieldValue": event.getParam('fieldValue')
        });
        appevt.fire();
    }
})

```
---

{{< figure src="/img/auraDomEventListener.png" title="Aura Wrapper" >}}

---

## Create your Aura Listener

The Aura Listener component is merely here for educational purposes, normally, you would actually have some sort 
of component that hadnles the event to trigger some kind of action. Here we will simply use the received evevnt parameters and display
them in a box.

## Listener Component Markup


```
<!-- auraListener.cmp -->
<aura:component implements="forceCommunity:availableForAllPageTypes">
    <aura:attribute name="receiver" type="List"/>
    <aura:attribute name="objReceived" type="Map" default="{}"/>
    <aura:handler event="selfService:caseCreateFieldChange" action="{!c.handleApplicationEvent}"/>
    
    <lightning:card title="AuraAppEventListener" iconName="custom:custom30">
        <aura:set attribute="actions">
            <span class="aura">Aura Component</span>
        </aura:set>
        <div class="slds-m-around_medium slds-align_absolute-center">
            <lightning:layout>
                <lightning:layoutItem size="5" class="slds-box">
                    
                    <aura:iteration items="{!v.receiver}" var="data">
                        <div class="slds-page-header data-row">
                            {!data.subject}
                        </div>
                      </aura:iteration>

                </lightning:layoutItem>
                <lightning:layoutItem size="8" class="slds-box">
                    
                    <aura:iteration items="{!v.receiver}" var="data">
                        <div class="slds-page-header data-row">
                            {!data.description}
                        </div>
                      </aura:iteration>

                </lightning:layoutItem>
            </lightning:layout>
        </div>
    </lightning:card>
</aura:component>

```

{{< figure src="/img/auraListener.png" title="Aura Listener" >}}


## Aura Listener Controller

In the Listener, we create a Set, in order to avoid duplicate key values, and then convert the set to an array to be able to 
work with the data in the component, since we did declare a List (Array) attribute, and not a Set.

```
		
({
    handleApplicationEvent : function(component, event, helper) {
        let arraySet = new Set(); 
        let objArray = component.get('v.receiver');
        let objReceive = component.get('v.objReceived');

        const field = event.getParam('modifiedField');
        const fieldValue =  event.getParam('modifiedFieldValue');

        objReceive[field] = fieldValue;
        arraySet.add(objReceive);
        objReceive = Array.from(arraySet);
        
        component.set('v.receiver', objReceive)

    }
})

```

[!Firing and receiving the event](http://recordit.co/j5kdCSi7UF)