# HomeFolderLinkcmdGlobal Extension

## Visual Studio Code

:fa-github:

[Visual Studio Code](https://code.visualstudio.com/)

!!! note
    This tutorial example is for 4.0

The global command bar in active workspace allows for generic links to be made that can help reduce clicks or input time to get to a certain location.

The HomeFolderLinkcmdGlobal does just this.  Instead of having to click home then home folder this module places a home folder command in the global toolbar that takes any user to their home folder no matter where they are in the client.  
 

The example also shows how to use javascript and services to get information required to build the url for the home folder.  The home folder itself is just a constructed URL based upon the users home folder UID.  The location is just a showObject location, so something like this:

        com.siemens.splm.clientfx.tcui.xrt.showObject?s_uid=gofF7D0qqd$DyB&uid=gofF7D0qqd$DyB

com.siemens.splm.clientfx.tcui.xrt.showObject is the location being used.  This location needs a uid input to display whatever object needs to be displayed though.  The UID for my home folder is the gofF7D0qqd$DyB.  So how do we get that uid from the client?  We need to use a service to find it.

Inside of the module.json you will find the actions associated with this command.  The action ‘activateHomeFolderLinkcmdGlobal’ is actually a service being called from some javascript.
 
So here we can see the action type as a JSFunction, this calls a javascript function from a certain file based on the two attributes method and deps.  Method defines the method you want to use, deps defines the file you want to use.  So this example specifically defines the method to be used as getProperties in the file located in soa/dataManagementService in your war file. (technically its within assets/js/soa/dataManagementService but it starts looking in the js folder initially)
 
So now we know the method and the file its located in so we can find its inputs 
exports.getProperties = function( uids, propNames ) 
So the inputs are uids and propNames for the uids.  We can provide that in the json as well:
 
We know the home folder property is a typedReference on the user object so getting the uid for that should be simple.  The input for uids should be the user uid which we can get through ctx like like shown above with ctx.user.uid.  We can easily see the ctx data if we run window._jsniInjector.service('appCtxService').ctx or download the available chrome dev tools for declarative(Link to the chrome dev tools if available or remove this).  The property on the user object we want is a TypedReference which will return the UID of the typedReference object, on the User object that property is home_folder like above.

Now that all of the inputs are solid we need to handle the output of the data and put it in a position for easy access when we build the url for the link.
 
Here we are going to take the output of the method above and create our own ctx.user.<something> that we can access when we click the link.  
 
So from above in outputData we defined a dataParseDefinition to call which is updateUserCtxForHomeFolder.  This dataParseDefinitions is taking the output format which will be a ViewModelObject from the typedReference home_folder property from the User object above and we are going to create that property under ctx.user so we have access to it.  The dataInput here is from the service return since we know there is only 1 object that should be returned here.  

Now that we have finished the service call and had a success with our return and updated a property on ctx.user so we have access to the home folder UID we can call a new action to generate the link:
 
This event occurs from the initial action ‘activateHomeFolderLinkcmdGlobal’ success.  So as long as the service and the ctx.user update is successful then we will call the event showHomeFolderLink:
 
This event is just going to call an action showHomeFolderLink:
 
In AW 4.0 there is a new actionType.  This actionType is  called Navigate which can help replace any javascript code that generates and executes a URL in the same tab.  The attributes are populated to determine where in AW you want to send the user.  For this example we are going to send them to showObject location which is the com_siemens_splm_clientfx_tcui_xrt_showObject above.  There are parameters we can apply to this as well since showObject has a UID.  From earlier we looked at how this is updating ctx.user props to allow us to use it later, this is where we use it.  The uid in navigationParams can be set based on what we did earlier, ctx.user.props.home_folder.dbValues[0] is now the uid of the users home_folder which we need to generate the URL.  cmdID and cmdArg are used for tiles actually in the values you can generate there.  For this example we can ignore them.

In the end here is what the final product does:
