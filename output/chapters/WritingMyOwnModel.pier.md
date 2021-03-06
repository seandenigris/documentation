

##1\.  Creating new basic widget


*Spec* provides for a large amount and wide variety of basic widgets\. 
In the rare case that a basic widget is missing, the 
*Spec* framework will need to be extended to add this new widget\.
In this section we will explain how to create such a new basic widget\.


We will first explain the applicable part of how the widget build process is performed\.
This will reveal the different actors in the process and provide a clearer understanding of their responsibilities\.
We then present the three steps of widget creation: writing a new model, writing an adapter, and updating or creating an individual UI framework binding\.



###1\.1\.  One step in the building process of a widget


The UI building process does not make a distinction between basic and composed widgets\.
Hence, at a specific point in the building process of a basic widget the default spec method of the widget is called, just as if it would be a composed widget\.
However in this case, instead of providing a layout for multiple widgets that comprise the UI, this method builds an adapter to the underlying UI framework\.
Depending of the underlying UI framework that is currently used, this method can provide different kind of adapters, for example an adapter for Morphic, or an adapter for Seaside, etc\.


The adapter, when instantiated by the UI model, will in turn instantiate a widget that is specific to the UI framework being used\.


For example, when using a List in the Morphic UI, the adaptor will be a MorphicListAdapter and it will contain a PluggableListMorph\.
This is this framework\-specific widget that will be added to the widget container\.


Figure 
[1\.1](#model_adapter_uielement) shows the relationship between those objects\.


<a name="model\_adapter\_uielement"></a>![model\_adapter\_uielement](figures/Model-Adapter-UIElement.png "Relationship between the model, the adapter, and the UI element")


###1\.2\.  The Model


The new model needs to be a subclass of 
**AbstractWidgetModel** and its name should be composed of the new basic widget concept, e\.g\. list or button, and of the word 
*Model*\.
The responsibility of the model is to store all the state of the widget\.
Examples of widget\-specific state are:


-  the index of a list
-  the label of a button
-  the action block for when a text is validated in a text field


The state is wrapped in value holders and kept in instance variables\.
For example, the code in 
[1\.1](#ex_value_holder) shows how to wrap the state 
`0` in a value holder and keep it as an instance variable\.
Value holders are needed because they are later used to propagate state changes and thus create the interaction flow of the user interface, as discussed in Section 
[¿?](#sec_heart_of_spec)\.




<a name="ex_value_holder"></a>**Storing state wrapped in a Value Holder in an instance variable**


    index := 0 asValueHolder.



For each instance variable that holds state three methods should be defined: the getter, the setter, and the registration method\.
The first two should classified in the protocol named 
*protocol* while the registration method should be in 
*protocol\-events*\.
For example, the code in 
[1\.2](#ex_mutators) shows the methods for the example code in 
[1\.1](#ex_value_holder)\. 




<a name="ex_mutators"></a>**Example of mutators for index**


    "in protocol: protocol"
    index
    	^index value
    
    "in protocol: protocol"
    index: anInteger
    	index value: anInteger
    
    "in protocol: protocol-events"
    whenIndexChanged: aBlock
    	index whenChangedDo: aBlock



The last step to define a new model is to implement a method 
`adapterName` at the class side\.
The method should be in the protocol named 
*spec* and should return a symbol\.
The symbol should be composed of the basic concept of the widget, e\.g\. list or button, and the word 
*Adapter* like 
**ListAdapter**\.


The communication from the UI model to the adapter is performed using the dependents mechanism\.
This mechanism is used to to handle the fact that a same model can have multiple UI elements concurrently displayed\.
In fact the message 
`change: with: ` is used to send the message 
*selector* with the arguments 
*aCollection* to the adapter\.




    Note: For Ben. I still don't understand it. Please explain to me tomorrow.




###1\.3\.  The Adapter


An adapter must be a subclass of 
**AbstractAdapter**\.
The adapter name should be composed of the UI framework name, e\.g\. Morphic, and the name of the adapter it is implementing, e\.g\. ListAdapter\.
The adapter is an object used to connect a UI framework specific element and the framework independent model\.


The only mandatory method for an adapter is 
`defaultSpec` on the class side\.
This method has the responsibility to instantiate the corresponding UI element\.


The example 
[1\.3](#ex_adapter_instanciation) shows how 
**MorphicButtonAdapter** instantiates its UI element\.




<a name="ex_adapter_instanciation"></a>**How MorphicButtonAdapter instantiates its UI element**


    defaultSpec
    	<spec>
    	
    	^ {#PluggableButtonMorph.
    			#color:. Color white.
    	    		#on:getState:action:label:menu:. 	#model. #state. #action. #label. nil.
    			#getEnabledSelector:. 				#enabled.
    			#getMenuSelector:.				#menu:.
    			#hResizing:. 						#spaceFill.
    			#vResizing:. 						#spaceFill.
    			#borderWidth:.						#(model borderWidth).
    			#borderColor:.						#(model borderColor).
    			#askBeforeChanging:.				#(model askBeforeChanging).
    			#setBalloonText:.					{ #model . #help}.
    			#dragEnabled:.						#(model dragEnabled).
    			#dropEnabled:.						#(model dropEnabled).	
    			#eventHandler:.					{	#EventHandler. #on:send:to:. #keyStroke.	#keyStroke:fromMorph:. #model	}}



Since the adapter is bridging the gap between the element of the UI framework and the model, the adapter also needs to forward the queries from the UI element to the model\.
Seen from the other way around: since the model is holding the state, the adapter is used to update the UI element state of the model\.


The methods involved in the communication from the model to the adapter as well as the methods involved in the communication from the adapter to the UI model should be in the protocol 
*spec protocol*\.
On the other hand the methods involved in the communication from the adapter to the UI element and vice versa should be categorized in the protocol 
*widget API*\.


To communicate with the UI element, the adapter methods use the method 
`widgetDo:`\.
This method executes the block provided as argument, which will only happen after the UI element has already been created\.


The example 
[1\.4](#ex_emphasis) shows how 
**MorphicLabelAdapter** propagates the modification of the emphasis from the adapter to the UI element\.




<a name="ex_emphasis"></a>**How MorphicLabelAdapter propagates the emphasis changes**


    emphasis: aTextEmphasis
    
    	self widgetDo: [ :w | w emphasis: aTextEmphasis ]




###1\.4\.  The UI Framework binding


The bindings is an object that is used to resolve the name of the adapter at run time\.
This allows for the same model to be used with several UI frameworks\.


Adding the new adapter to the default binding is quite simple\.
It requires to update two methods: 
`initializeBindings` in 
**SpecAdapterBindings** and 
`initializeBindings` in the framework specific adapter class, e\.g\. 
**MorphicAdapterBindings** for Morphic\.




    Note: For Ben: Give an example here.



Once this is done, the bindings should be re\-initialized by running the following snippet of code: 
`SpecInterpreter hardResetBindings`\.


For creating a specific binding, the class 
**SpecAdapterBindings** needs to be overriden as well as its method 
`initializeBindings`\.
It can then be used during a spec interpretation by setting it as the bindings to use for the 
**SpecInterpreter**\.
The example 
[1\.5](#ex_setting_bindings) shows how to do so\.




<a name="ex_setting_bindings"></a>**How to set custom bindings**


    SpecInterpreter bindings: MyOwnBindingClass new.



Note that the 
**SpecInterpreter** bindings are reset after each execution\.
