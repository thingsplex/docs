---
layout: default
title: Data operations
parent: Flows
grand_parent : Thingsplex UI
nav_order: 4
---

# Data operations

## Set variable

You can make as many variables as you like in your flows. Variables can be used to save the state of your devices, values read from a REST Action, mode of your site or similar. In the **conditions** section there was mentioned that it is possible to trigger **if** mode is not vacation. To check what mode you are in, you can use a global variable. This can be done by using a **Home event trigger** that triggers whenever mode is changed, that sends its value directly in to a **set_variable** node that saves the string value in a variable. First we need to make the variable. 

![Variables](img/variables.png)

To make a variable, click on **Add** in the set_variable node. Then choose a name, description, and type. In this case the type should be **string** as we want to save the mode. If you save the variable in a different flow than where you want to use it, you need to make sure it is **Global** by ticking the **in memory** box. Your variable configuration should then look like this:

![Set variables](img/set-variables.png)


## Transform node

Transform node performs more advanced data transformations . The result is assigned to *Result variable*
**Supported transformation types :** 
* Calculate 
* Map 
* JsonPath 
* XPath
* Tempelate 

### Calculate 
The transformation type executes expression set in *Expression* field and saves result into *Result variable*.
Epression supports all local and global variables using their normal names and input variable as **input** and Left variable as **variable** .
Assuming we have variables *address* , expression can be `address+100` , result will be assigned to *Result variable*

### Map


### JsonPath

### Xpath

### Template