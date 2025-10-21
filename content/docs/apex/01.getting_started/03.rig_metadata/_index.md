---
date: '2025-10-19T22:17:16+11:00'
title: 'Guides and Metadata'
weight: 4
---


Guides allow you to build additional joint on top of the deformation skeleton.

 
## Skeleton Metadata Overview

### Tags
Tags are used to add metadata to a rig, allowing it to be setup  procedurally. For example you could tag a characters arms, with the tags #arm and #left or #arm and #right. This would allow you to apply a rig component, for example an IK chain, to those joints without having to specify the joint names. This makes it easy to quickly and setup a rig on characters which may have different naming conventions or joint structures. 

Tags are used heavily for both the **Autorig Builder** and **Autorig Component** nodes to specify how the component attaches to a rig. 

In SOPs you can use the **Attribute Adjust Tags** node, which allows you to select joints and tag them in the viewport.

As tags are simply string arrays attribute, you can set them using VEX or VOPs. For example, to tag joints based on the side of a character:

```python
s[]@tags = {};
@P.x > 0.01 ? append(s[]@tags, "left") : 0 ;
@P.x < -0.01 ? append(s[]@tags, "right") : 0 ;
```

And as it is an attribute the usual tools for manipulating them are available to you in SOPs. 

> [!TIP]
> Tags are a concept that it worth getting familiar with; in this case we are defining tags on the guide geometry, however tags can also be use to tag nodes and ports in APEX graphs. This allows you to procedurally find nodes or ports when building graphs and is something we will heavily use later when building APEX graphs. 

### Segments
Segments are tags, and are defined using the same `s[]@tags` string array. The only difference in that segments are used to define segments of a rig; for example and arm, or spine. 

In the example below we will use them to define the arm and leg segments and attach IK chains to them use the **Autorig Component** node. 

### Propterties
Another attribute that can be defined on the skeleton are properties. These can be used to define a range of properties on the controls generated from the shape, size and colour of the controller, to which handles get promoted for the animator to use. Properties are defined by a dictionary point attribute on the joint. 

For example, the vex below will set the colour, shape and promoted the t and s handles:

``` python
dict d;
d['color'] = set(1.0, 1.0, 0.0);
d@properties['control'] = d;
d@properties['promote'] = 't s';
d@properties['shape'] = 'box';
```

Which would give use this dictionary:

``` python
{
    "control":{
        "color":[1,1,0]
    },
    "promote":"t s",
    "shape":"box"
}
```

Another way to setup controls would be using the **APEX Configure Controls** SOP, or on the *Transform Objects* on the rig (which is where these attributes are ultimately used). It can be useful to define the controls on the guide geometry in SOPs. There is no correct way here, but rather whatever way best suites the desired workflow. 

## Example 1: Adding Tags
![](/apex/img/getting_started.tags_overview.png)
As a quick example of how the Guides.skel, tags and joint metadata can be used, we’re going to quickly walk through how to apply tags then add a basic IK chain to the Capybara.

### Example File
Download the example file here: [getting_started.tags.hiplc](/apex/img/getting_startes.simple_rig.hiplc){{< icon "document-download" >}}


### Adding Tags
{{< video src="adding_tags" >}}
To add tags in viewport, display the *set tags* node.Then in the viewport select the joints you want to add the tags to and press ‘a’. Type in the name of the tag you want to add and press enter. For this example we want to create two tags:

- First select the finger joints on both arms and tag them as ‘fk’. 
- Secondly, select the arm, elbow and hand joints on the the left arm and tag them as ‘l_arm’ and tag the same joints on the right arm and tag them ‘r_arm’. 


### Using the FK Tag
![](/apex/img/getting_started.tags_fk.png)
The first thing we are going to do is to use the fk tags to promote the rotation for the finger joints (but not the arm joints as they’ll use an ik component). To do this we use a *fk transform* component on the **Autorig Builder**. 

As we saw in the previous example, the *fk transform* creates joints for all of the joints on the skeleton. To get this working with our guide skeleton and tag, there are two parameters we need to change:

1. Under *Source* set the *Skeleton* to `Guides.skel`. This ensures it will use our tagged joints rather than the base skeleton.
2. In the *Controls* tab, set the *Promote R* parameter to `#fk`. This means that it will only promote the joints with the fk tag. Looking in the viewport you should now only see controls for the finger joints.

### Using the Arm Segment Tag
![](/apex/img/getting_started.tags_ik.png)
Next we want to use the arm segment we setup to add an ik chain to our arms. This is pretty straightforward. 

1. Go to the *fk* **Autorig Component** node. From the *Component Source* parameter at the top of the node select *Mulit IK*. In the viewport you should see some controllers floating around. The component doesn’t know where to attach them and its guess is a bit off in this case. 
2. On the *Driven* tab there is a parm for segments. You could type `r_arm l_arm`, but the parameter allows for pattern matching, so we can just type `*_arm` to get the ik chains created for both arms. 

## Example 2: Control Options and Guides
![](/apex/img/getting_started.tags_properties.png)
This second example uses the same file as the previous one. 

As mentioned above, there's a variety of metadata that can be attached to each joint (point) on the skeleton. This can control the shape or the colour of the controller, as well as setting limits on the controller and defining the channels which get promoted. 

In the example file we are:

1. Copying the elbow joint and offsetting it, to give us our pole vector controller.
2. Creating a dictionary which defines the look of the controller and promotes just the translate channel. 
3. Finally we mirror the pole vector joint and parent it to the hand joint. 
 
### Joint Metadata Setup
The vex code for setting up the properties dictionary in the *set_properties* attribute wrange is:

```python
// Set the Control Dictionary
dict control = set(
                    'shapeoverride', 'box_wires',
                    'color', set(0,1,0)
                    );
                    
// Set properties dictionary
d@properties = set(
                    'control', control,
                    'max_lock:t', set(0.5,0.5,0.5),
                    'min_lock:t', set(-0.5,-0.5,-0.5), 
                    'promote', 't'
                    );
```

Which generates a dictionary which should look like this:

```python
{
    "control":{
        "color":[0,1,1],
        "shapeoverride":"box_wires"
    },
    "max_lock:t":[0.5,0.5,0.5],
    "min_lock:t":[-0.5,-0.5,-0.5],
    "promote", "t"
}
```

In the animate state of the **Autorig Component** there should be two large wireframe boxes, and if you try and move them they will be limited to the range set in the properties dictionary. Additionally, rotating the hand joints will rotate the pole vector controllers; the hierarchy we defined in the Guide skeleton has been brought across to our rig.
