<!--
parent: QTI
created_at: '2014-04-08 18:02:29'
updated_at: '2015-03-12 10:01:42'
authors:
    - 'Somsack Sipasseuth'
tags:
    - QTI
-->

Qti Item Creator
================



Lexicon
-------

-   **element or qti element** : refers to an instance of a qti class defined in [the qti 2.1 standard](http://www.imsglobal.org/question/qtiv2p1/imsqti_infov2p1.html)
-   **widget** : it represent the common structure for the qti element that is being edited (see step3 of the tutorial for more details)
-   **state** : the state (or mode) define specific behaviours for a widget (see step3 of the tutorial for more details)

Files structure
---------------

The source of the Qti Creator is located in `{tao_root}/taoQtiItem/views/js/qtiCreator/test/renderer/`.<br/>

Any later reference of “qtiCreator/” in this wiki page means `{tao_root}/taoQtiItem/views/js/qtiCreator/`.

-   **qtiCreator/core** : no longer used (deprecated, will be removed by the final release)
-   **qtiCreator/css** : not currently used
-   **qtiCreator/editor** : lib for the main editor view
-   **qtiCreator/helper** : helper for the main editor view
-   **qtiCreator/lib** : third party js libraries
-   **qtiCreator/model** : the model of qti element that will be used to reprent a qti item and all its composing element, it extends the qtiItem/core one
-   **qtiCreator/preview** : code specific for the preview mode
-   **qtiCreator/renderers** : declare the renderer for each individual qti element : the responsibility of the renderer is limited to setting the html template and the proper instanciation of the element creator widget
-   **qtiCreator/test** : all js unit test are located here
-   **qtiCreator/tpl** : the template directory
-   **qtiCreator/widgets** : contains the widgets and their states which define all behaviour of the qti element editor

Tutorial : implement a new qti item creator widget
--------------------------------------------------

### step 0: define the behaviour you want to implement

First, please have a look to the existing implementations so you can have a better feel of expecting behaviour.<br/>

The test can be found here: `{tao_root}/taoQtiItem/views/js/qtiCreator/test/renderer/`

Please note that the following actions are required:

In the question state:

-   add choice (with add choice button)
-   remove choice (via mini-toolbar)
-   edit choice content, using ckeditor for html content or simple contenteditable element for plain text
-   edit prompt (for blockInteraction) using ckeditor
-   show additional property form for the interaction
-   show additional property form for the choice

In the answer state:

-   show response property form
-   [correct state]() only the correct answer need to be defined. The item author should be able to edit the correct response as if she/he is in the delivery mode : the **commonRenderer.render() is reused here**
-   [map (response) state]() there should be a way for item author to “map” a response to a “score”. Here, the correct answer can also be edited. However defining the correct response in the “map response” state is optional so should be hidden initially and available on user’s demand.

After integration, your interaction widget (and form) will look like this:

Sleep state:<br/>

![](../resources/item-creator-1-main.png)

Question state:<br/>

![](../resources/item-creator-2-question-mode.png)

Choice state:<br/>

![](../resources/item-creator-3-choice-mode.png)

Answer state:<br/>

![](../resources/item-creator-4-answer-mode.png)

The template will be available in the style-guide.

Once the behaviour clrealy defined, we can get started !

### step 1: extending the qti item model for authoring

**Location** : `{tao_root}/taoQtiItem/views/js/qtiCreator/model`

It defines the qti element classes for authoring by extending the base qti item classes from `{tao_root}/taoQtiItem/views/js/qtiItem/core`.<br/>

It adds some additional methods for authoring purpose (createChoice, createElements, etc) and set reasonable default attributes values.

Tip for developers : if you are implementing a new interaction and its choice for the item creator, you need first to ensure that the model is properly defined here. In practice, just follow the example implementation, (e.g. qtiCreator/model/interactions/ChoiceInteraction.js) and implement the 3 methods :

-   getDefaultAttributes
-   afterCreate
-   createChoice




    define([
        'lodash',
        'taoQtiItem/qtiCreator/model/mixin/editable',
        'taoQtiItem/qtiCreator/model/mixin/editableInteraction',
        'taoQtiItem/qtiItem/core/interactions/ChoiceInteraction',
        'taoQtiItem/qtiCreator/model/choices/SimpleChoice'
    ], function(_, editable, editableInteraction, Interaction, Choice){
        var methods = {};
        _.extend(methods, editable);
        _.extend(methods, editableInteraction);
        _.extend(methods, {
            getDefaultAttributes : function(){
                return {
                    'shuffle' : false,
                    'maxChoices' : 0,
                    'minChoices' : 0,
                    'orientation' : 'vertical'
                };
            },
            afterCreate : function(){
                this.createChoice();
                this.createChoice();
                this.createChoice();
                this.createResponse({
                    baseType:'identifier',
                    cardinality:'multiple'
                });
            },
            createChoice : function(){
                var choice = new Choice();

                this.addChoice(choice);

                var rank = _.size(this.getChoices());

                choice
                    .body('choice' + ' #' + rank)
                    .buildIdentifier('choice');

                if(this.getRenderer()){
                    choice.setRenderer(this.getRenderer());
                }

                $(document).trigger('choiceCreated.qti-widget', {'choice' : choice, 'interaction' : this});

                return choice;
            }
        });
        return Interaction.extend(methods);
    });

For the choices, you only need to define the getDefaultAttributes() and applying the editable mixin\
You can follow the example here: `qtiCreator/model/choices/SimpleChoice.js`


    define(['lodash', 'taoQtiItem/qtiCreator/model/mixin/editable', 'taoQtiItem/qtiItem/core/choices/SimpleChoice'], function(_, editable, Choice){
        var methods = {};
        _.extend(methods, editable);
        _.extend(methods, {
            getDefaultAttributes : function(){
                return {
                    'fixed' : false,
                    'showHide' : 'show'
                };
            }
        });
        return Choice.extend(methods);
    });

### step 2 : define the qtiCreatorRenderer

**Location:** `taoQtiItem/views/js/qtiCreator/renderers`

As for qtiCommonRenderer, the renderer for each qti element (interaction, choices, response, image etc.) are located in the “renderers” subdirectory .

The goal is to reuse what has been done for the commonRenderer, so for the choice interaction creator renderer, we have this code:


    define([
        'lodash',
        'taoQtiItem/qtiCommonRenderer/renderers/interactions/ChoiceInteraction',
        'taoQtiItem/qtiCreator/widgets/interactions/choiceInteraction/Widget'
    ], function(_, ChoiceInteraction, ChoiceInteractionWidget){

        var CreatorChoiceInteraction = _.clone(ChoiceInteraction);

        CreatorChoiceInteraction.render = function(interaction, options){

            return ChoiceInteractionWidget.build(
                interaction,
                ChoiceInteraction.getContainer(interaction),
                this.getOption('interactionOptionForm'),
                this.getOption('responseOptionForm'),
                options
            );
        };

        return CreatorChoiceInteraction;
    });

It basically uses everything defined in the commonRenderer but overwrite the render method.<br/>

The render method here just builds the choice interaction widget. Presentation of qti creator widgets will be detailed in a later section.

For a creator choice renderer, we are doing the same : loading its commonRenderer and overwrite the render method.


    define([
        'lodash',
        'taoQtiItem/qtiCommonRenderer/renderers/choices/SimpleChoice.ChoiceInteraction',
        'taoQtiItem/qtiCreator/widgets/choices/simpleChoice/Widget'
    ], function(_, SimpleChoice, SimpleChoiceWidget){

        var CreatorSimpleChoice = _.clone(SimpleChoice);

        CreatorSimpleChoice.render = function(choice, options){

            SimpleChoiceWidget.build(
                choice,
                SimpleChoice.getContainer(choice),
                this.getOption('choiceOptionForm'),
                options
            );
        };

        return CreatorSimpleChoice;
    });

**Note** : an interaction widget require one more argument compared to a choice : the ‘responseOptionForm’

**Important notice** : please do not forget to declare the new renderer in the\
`taoQtiItem/views/js/qtiCreator/renderers/config.js` otherwise, the commonRenderer will still be called instead of this one/

### step 3 : define the widget and its states

**location** : `taoQtiItem/views/js/qtiCreator/renderers`<br/>

Because of the complexity of the authoring widgets, it would be difficult to manage everything in a single render file like we are doing in the CommonRenderers. Instead everything is managed by qti creator Widgets. The behaviour of every widgets change according to their states. All interactions and choices share the same set of states.<br/>

This is to ensure behaviour consistency across all interactions and other qti elements implementations and decrease overall complexity.

#### widget

The widget define the common structure for the qti element that is being edited.<br/>

The stack of states will bring and enrich the behaviour as needed.

Instanciation and usage:<br/>

A widget is only instanciated in the qti element renderer (see previous “renderer declaration section”)

Definition

Let’s start with an example:


    define([
        'taoQtiItem/qtiCreator/widgets/interactions/Widget',
        'taoQtiItem/qtiCreator/widgets/interactions/choiceInteraction/states/states'
    ], function(Widget, states){

        var ChoiceInteractionWidget = Widget.clone();

        ChoiceInteractionWidget.initCreator = function(){

            Widget.initCreator.call(this);

            this.registerStates(states);

            this.$container.find('.qti-choice > .pseudo-label-box input').prop('disabled', 'disabled');
        };

        return ChoiceInteractionWidget;
    });

Here we are creating the choice interaction creator widget by extending the base “qtiCreator/widgets/interactions/Widget” one.<br/>

Upon instanciation, the initCreator() method will be called. Variables and default behaviours can be defined in the initCreator() method. Please note here that the parent Widget.initCreator() is also called and the registerStates(states) is absolutely mendatory. After instanciation, the state of the widget will initially be set to “sleep”.<br/>

The call sequence is thus this:

`creatorRenderer.render() -> widget.build() -> widget.initCreator() -> widget.changeState('sleep')`

From here, all behaviour will be managed by the different states that have been registered in the widget.

#### states

The states are defined as follow.<br/>

![](../resources/taoQtiCreatorStates.png)

Developer note : some states are common to all widgets (sleep, deleting, active, moving). They are marked in blue-gray in the diagram above. You usually don’t need to change anything in those states.<br/>

However, for every interaction widget, you have to implement the question at least, plus the states choice, answer, correct and map states if applicable (they are marked in orange-yellow in the diagram). Please refer to an existing implementation as an example to implement this.<br/>

For an interaction where a response declaration is not required (such as mediaInteraction or uploadInteraction), there is no need to define an answer state.

State hierarchy :<br/>

The states are hierachical. When you are in the question state, you will also inherit the behaviour of the active state. When you are editing a response in the correct state, you will inherit the behaviour of both the active and answer states. The following diagram illustrates a stack of states when an interaciton or a choice is in the correct state.<br/>

![](../resources/taoQtiCreatorStatesStacks.png)

#### defining a widget state

There are 3 ways to define a state for a qti creator widget. All of them must be done with the state factory: qtiCreator/widgets/states/factory.js

1 . Define a state from scratch by giving its name, (optional) superStates, and required init() and exit() callback methods.


    define(['taoQtiItem/qtiCreator/widgets/states/factory'], function(stateFactory){
        return stateFactory.create('question', ['active'], function(){
        console.log('I am entering the quesiton state now');
        },function(){
        console.log('I am leaving the quesiton state now');
        });
    });

2\. Define a state from another state, without extending it. Useful when you want to reuse everything defined in the parent state but just need to overwrite a few methods.<br/>

In the example below, the generic question state of the abstract block interaction is being extended to create the question state of the choice interaction. Only the initForm() method has overwritten.


    define([
        'taoQtiItem/qtiCreator/widgets/states/factory',
        'taoQtiItem/qtiCreator/widgets/interactions/blockInteraction/states/Question'
    ], function(stateFactory, Question){

        var ChoiceInteractionStateQuestion = stateFactory.create(Question);

        ChoiceInteractionStateQuestion.prototype.initForm = function(){

            console.log('I am using everything defined in the parent but just overwriting a method here');
        };

        return ChoiceInteractionStateQuestion;
    });

3\. Define a state from another state, by extending it in a OOP-kind sense. Useful when you want to add new behaviour while keeping existing one.


    define([
        'taoQtiItem/qtiCreator/widgets/states/factory',
        'taoQtiItem/qtiCreator/widgets/interactions/blockInteraction/states/Question'
    ], function(stateFactory, Question){

        var ChoiceInteractionStateQuestion = stateFactory.extend(Question, function(){
            console.log('The init method of the parent Question state will be called before me');
        }, function(){
            console.log('The exit method of the parent Question state will be called after me');
        });

        ChoiceInteractionStateQuestion.prototype.initForm = function(){

            console.log('I am using everything defined in the parent but just overwriting a method here');
        };

        return ChoiceInteractionStateQuestion;
    });

According to your need (whever you need to define new behaviours from the parent or not), the second or third method should be used. Doing this will ensure an overall consistency accros all widget and states!

#### variable access in your states

Upon instanciation, a state is given the reference of the widget it represents.<br/>

You can access anything you want in the state in this.widget:


    define(['taoQtiItem/qtiCreator/widgets/states/factory'], function(stateFactory){
        return stateFactory.create('active', function(){

            console.log('the widget is in the active state now');

            var _widget = this.widget;


            console.log('this is the qti element being edited', _widget.element);
            console.log('this is the form container where I should append form to edit its properties', _widget.$form);
            console.log('this is the widget container everything is going to happen', _widget.$container);
            console.log('this is the original container from where the widget is being build', _widget.$original);

        },function(){

            console.log('the widget is no longer in the active state');
        });
    });

### step 4 : create the option forms

The widgets in the center panel of the item creator allow editing the item, interaction and choice contents. Some properties can also be edited (pin/shuffle).<br/>

However, most of the meta, options are to be positioned in the right side bar in an appropriate form.

-   every interaction has a from representing its options
-   every choice has its own form too
-   most of interaction will be sharing the same response processing form - string interactions have more options because they allow more than one baseType (string, integer, float)

#### define the template

Using the qti 2.1 standard, please make note of all properties required for the qti element you are working on.<br/>

Then use either the style guide or an existing example to create your html template. Location: qtiCreator/tpl/forms/<br/>

You may note the use of specific validator to easily control the input value : in the following example, the identifier must not be empty (meaning it is required), and must also be a string that match the qti identifier format and must also not be already used.



        {{__ "identifier"}}

        The identifier of the choice. This identifier must not be used by any other choice or item variable



#### Append the form to the DOM

Within a state, you have access to the <br/>
$form by this.widget.<br/>
$form property, which is the container of the form.<br/>

After you leave the state, this container will automatically be emptied. You only have to worry about its initialization as in the example below:


            this.widget.$form.html(formTpl({
                serial:_widget.element.getSerial(),
                identifier:_widget.element.id()
            }));

This exemple comes from /qtiCreator/widgets/choices/simpleChoice/states/Choice.js

#### Init the data binding

You need to use taoQtiItem/qtiCreator/widgets/helpers/formElement to bind modification of any declared element of your form to callback function.<br/>

This callback will only be executed when the input is valid. To set your form element validation, please have a look on the exemple of the “define the template” section


            formElement.initDataBinding(this.widget.$form, this.widget.element, {
                identifier : identifierHelper.updateChoiceIdentifier
            });

Here, we want the callback identifierHelper.updateChoiceIdentifier to be executed when the form element with [name=identifier] changes and the value of which is valid.<br/>

You may want to reuse this callback whenever a choice identifier is being edited to sync its values with response declarations.<br/>

But in practice you can define any callback you want to save the property value, like in this simple example below:


            formElement.initDataBinding(this.widget.$form, this.widget.element, {
                identifier : function(element, value, name){
                     var choice = element;
                     choice.attr(name, value);
                }
            });

As in JQuery, the DOM element is passed as the context of the callback :


            formElement.initDataBinding(this.widget.$form, this.widget.element, {
                identifier : function(element, value, name){
                     //you can have access to any properties, data-attributes of the DOM element
                     var $elt = $(this);
                     console.log($elt.attr('name'));
                }
            });

Working with the Qti Creator Item Model
---------------------------------------

We are using the model we defined in the step 1 of the tutorial.

### Editing API

When you are inside a widget or a state, you have access to the element you are currently working on.<br/>

The goal is to modify this object : modify attributes, add choice, modify a choice content, modify the body (if applicable), etc.<br/>

Every modification brought to the object will be serialized and saved as qti XML to the server via the qtiXmlRenderer.<br/>

Please use the api and not idrect access to object because those methods trigger the right events and following this api will help implementing undo/redo methods.

The main methods you may need are:

#### get and set attribute value


    interaction.attr('maxChoices', 3);

    var maxChoices = interaction.attr('maxChoices');
    console.log('maxChoices');//output: 3

#### create a choice for an interaction


    var interaction = new ChoiceInteraction();

    interaction.createChoice();
    interaction.createChoice();
    interaction.createChoice();

    var choices = interaction.getChoices();
    console.log(_.size(choices));//output: 3

#### get and set container body


    item.body('my qti item content');
    var itemBody = item.body();
    console.log(itemBody);//output: "my qti item content"

#### create elements in the container body


    item1.createElements('My QTI Item{{choiceInteraction:new}}', function(newElts){

            console.log(_.size(newElts));//output 1

            var interactions = this.getInteractions();
            console.log(_.size(interactions));//output 1
    });

### Events

When the model is modified, an event is trigger at the document level.

#### list of events:

They all are triggered in the same namespace “.qti-widget”

##### attributeChange

Triggered by `element.attr(name, value)`<br/>

see any interactions in qtiCreator/model/mixin/editable.js

##### deleted

Triggered by `element.remove()`<br/>

see any interactions in qtiCreator/model/mixin/editable.js

##### choiceCreated

Triggered by `interaction.createChoice()`<br/>

see any interactions in qtiCreator/model/interactions/

##### containerBodyChange

Triggered by `interaction.body()`<br/>

see any interactions in qtiItem/core/Container.js

##### choiceTextChange

Triggered by `interaction.createChoice()`<br/>

see any interactions in qtiCreator/model/choices/TextVariableChoice.js

##### responseTemplateChange

Triggered by `responseDeclaration.setTemplate()`<br/>

See qtiCreator/model/variables/ResponseDeclaration.js

##### correctResponseChange

Triggered by `responseDeclaration.setCorrect()`<br/>

See qtiCreator/model/variables/ResponseDeclaration.js

##### mapEntryChange

Triggered by `responseDeclaration.setMapEntry()`<br/>

See qtiCreator/model/variables/ResponseDeclaration.js

##### mapEntryRemove

Triggered by `responseDeclaration.removeMapEntry()`<br/>

See qtiCreator/model/variables/ResponseDeclaration.js

##### mappingAttributeChange

Triggered by `responseDeclaration.setMappingAttribute()`<br/>

See qtiCreator/model/variables/ResponseDeclaration.js

##### metaChange

Triggered by `editable.data(key, value)`<br/>

See qtiCreator/model/mixin/editable.js

#### listen to those events

You may want to listen to any modification mode to the qti element.<br/>

There is a simple way to do that via a method of the widget.<br/>

Important notice: by using this method, the event listener will have the same lifespan as the current state. If you bind an event in the *answer* state, it will still be active in a substate like *correct* or *mode*.<br/>

When you leave the answer state (say, you are goind to the *question* mode again), the listener will be automatically remove.

An example can be found in qtiCreator/widgets/interactions/states/Answer.js:



    //listen to any responseTemplateChange event
    _widget.on('responseTemplateChange', function(data){
           //if the template changes, forward the modification to a helper
           answerStateHelper.forward(_widget);

           //you can dump the data sent in the event and see what it contains
           console.log(data);
     });

Implementation example :
------------------------

Following the tutorial above, here is a sample skeleton for the mediaInteraction. (the uploadInteraction, which has no choices too, can follow the same example).

### *qtiCreator/model/interactions/MediaInteraction.js*


    define([
        'lodash',
        'taoQtiItem/qtiCreator/model/mixin/editable',
        'taoQtiItem/qtiCreator/model/mixin/editableInteraction',
        'taoQtiItem/qtiItem/core/interactions/MediaInteraction'
    ], function(_, editable, editableInteraction, Interaction){
        var methods = {};
        _.extend(methods, editable);
        _.extend(methods, editableInteraction);
        _.extend(methods, {
            getDefaultAttributes : function(){
                return {
                    autostart : false,
                    minPlays : 0,
                    maxPlays : 0,
                    loop : false
                };
            },
            afterCreate : function(){
                //function that will immediately be called after its creation in the item creator

                //nothing special to do here ?
            },
            createChoice : function(){
                throw 'mediaInteraction does not have any choices';
            }
        });
        return Interaction.extend(methods);
    });

Then I added a new entry in *qtiCreator/model/qtiClasses.js* to declare this new element in the qti item model for the qtiCreator:


    'mediaInteraction' : 'taoQtiItem/qtiCreator/model/interactions/MediaInteraction'

### *qtiCreator/renderers/interactions/MediaInteraction.js*


    define([
        'lodash',
        'taoQtiItem/qtiCommonRenderer/renderers/interactions/MediaInteraction',
        'taoQtiItem/qtiCreator/widgets/interactions/mediaInteraction/Widget'
    ], function(_, MediaInteraction, MediaInteractionWidget){

        var MediaInteraction = _.clone(MediaInteraction);

        MediaInteraction.render = function(interaction, options){

            return MediaInteractionWidget.build(
                interaction,
                MediaInteraction.getContainer(interaction),
                this.getOption('interactionOptionForm'),
                this.getOption('responseOptionForm'),//note : no response required...
                options
            );
        };

        return MediaInteraction;
    });

Then I added a new entry to *qtiCreator/renderers/config.js* to declare this new element renderer for the qtiCreator:


    'mediaInteraction' : 'taoQtiItem/qtiCreator/renderers/interactions/MediaInteraction'

### *qtiCreator/widgets/interactions/mediaInteraction/Widget.js*


    define([
        'taoQtiItem/qtiCreator/widgets/interactions/Widget',
        'taoQtiItem/qtiCreator/widgets/interactions/mediaInteraction/states/states'
    ], function(Widget, states){

        var MediaInteractionWidget = Widget.clone();

        MediaInteractionWidget.initCreator = function(){

            //first of all, register state:
            this.registerStates(states);

            //call parent init function:
            Widget.initCreator.call(this);

            //do some initialization, if needed:
            this.functionToInitMyInteractionCreatorWidget();
        };

        MediaInteractionWidget.functionToInitMyInteractionCreatorWidget = function(){
            //put some code here...
        };

        return MediaInteractionWidget;
    });

### *qtiCreator/widgets/interactions/mediaInteraction/states/states.js*


    define([
        'taoQtiItem/qtiCreator/widgets/states/factory',
        'taoQtiItem/qtiCreator/widgets/interactions/blockInteraction/states/states',
        'taoQtiItem/qtiCreator/widgets/interactions/mediaInteraction/states/Question',
        'taoQtiItem/qtiCreator/widgets/interactions/mediaInteraction/states/Sleep'
    ], function(factory, states){
        //the mediaInteraction state bundle contains 2 custom states Question and Sleep
        //the third argument of createBundle() enable us to exclude the answer, correct and map states from the inherited blockInteraction states bundle
        return factory.createBundle(states, arguments, ['answer', 'correct', 'map']);
    });

### *qtiCreator/widgets/interactions/mediaInteraction/states/Answer.js*


    define([
        'taoQtiItem/qtiCreator/widgets/states/factory',
        'taoQtiItem/qtiCreator/widgets/interactions/states/Answer',
        'taoQtiItem/qtiCreator/widgets/interactions/helpers/answerState'
    ], function(stateFactory, Answer, answerStateHelper){

        //does not much sense in mediaInteraction (same remark for uploadInteraction)
        //I would recommend disabling the answer button completely and remove this file if no longer required

        var MediaInteractionStateAnswer = stateFactory.extend(Answer, function(){

            var _widget = this.widget;

            //forward to one of the available sub state, according to the response processing template
            answerStateHelper.forward(_widget);

        }, function(){

            var _widget = this.widget;

        });

        return MediaInteractionStateAnswer;
    });

### *qtiCreator/widgets/interactions/mediaInteraction/states/Question.js*


    define([
        'taoQtiItem/qtiCreator/widgets/states/factory',
        'taoQtiItem/qtiCreator/widgets/interactions/blockInteraction/states/Question',
        'taoQtiItem/qtiCreator/widgets/helpers/formElement',
        'tpl!taoQtiItem/qtiCreator/tpl/forms/interactions/media'
    ], function(stateFactory, Question, formElement, formTpl){

        var MediaInteractionStateQuestion = stateFactory.extend(Question);

        MediaInteractionStateQuestion.prototype.initForm = function(){

            var _widget = this.widget,
                $form = _widget.$form,
                interaction = _widget.element;

            //initialize your form here, you certainly gonna need to modify it:

            //append the form to the dom (this part should be almost ok)
            $form.html(formTpl({

                //tpl data for the interaction
                autostart : !!interaction.attr('autostart'),
                loop : !!interaction.attr('loop'),
                minPlays : parseInt(interaction.attr('minPlays')),
                maxPlays : parseInt(interaction.attr('maxPlays')),

                //tpl data for the "object", this part is going to be reused by the "objectWidget", http://www.imsglobal.org/question/qtiv2p1/imsqti_infov2p1.html#element10173
                data:interaction.object.attr('data'),
                type:interaction.object.attr('type'),//use the same as the uploadInteraction, contact jerome@taotesting.com for this
                width:interaction.object.attr('width'),
                height:interaction.object.attr('height')
            }));

            formElement.initWidget($form);

            //init data change callbacks
            var callbacks = formElement.getMinMaxAttributeCallbacks(this.widget.$form, 'minPlays', 'maxPlays');
            callbacks['autostart'] = formElement.getAttributeChangeCallback();
            callbacks['loop'] = formElement.getAttributeChangeCallback();
            callbacks['data'] = function(interaction, attrValue, attrName){
                //some callback method when the input has been validated

                //something like:
                interaction.attr(attrName, attrValue);
            };
            //and so on for the other attributes...

            formElement.initDataBinding($form, interaction, callbacks);
        };

        return MediaInteractionStateQuestion;
    });

### *qtiCreator/tpl/forms/interactions/media.tpl*



          TO BE COMPLETED : add "object" editor (see : http://www.imsglobal.org/question/qtiv2p1/imsqti_infov2p1.html#element10173)






            {{__ "autostart"}}



            The autostart attribute determines if the media object should begin as soon as the candidate starts the attempt (checked) or if the media object should be started under the control of the candidate (unchecked).







            {{__ "loop"}}



            The loop attribute is used to set continuous play mode. In continuous play mode, once the media object has started to play it should play continuously (subject to maxPlays).




        {{__ "Allowed number of choices"}}


                The minPlays attribute indicates that the media object should be played a minimum number of times by the candidate.
                The maxPlays attribute indicates that the media object can be played at most maxPlays times - it must not be possible for the candidate to play the media object more than maxPlay times. A value of 0 (the default) indicates that there is no limit.




            Min



            Max





