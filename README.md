# PedDistancingGoTo Demo Model

## Summary

This project (repository) includes an AnyLogic simulation model which

  * defines a custom Pedestrian Library block which implements movement with
    attempts at maintaining a social distance;

  * includes a demo model which illustrates use of this block;

  * includes a Library element which allows the custom block to be exported as
    an AnyLogic library if desired (via a licensed AnyLogic Professional
    client), though we are not claiming this is an 'off-the-shelf' usable
    library for implementing social distancing in models (see later).

### AnyLogic Version

The model requires AnyLogic V8.5.2 or later.

### Licensing

It is released under a permissive MIT license (see `LICENSE.txt`).

## Background

With the recent COVID-19 pandemic, there has been a lot of focus on maintaining
social distance as a transmission prevention measure, with all the related
issues of how to manage that in the context of businesses or social
environments.

At DSE Consulting, we had a few clients enquiring about ways to explicitly model
this using AnyLogic, and it was something we'd already been interested in to
help the simulation community.

Within AnyLogic, there are roughly three common techniques used for moving
agents round in space that are most relevant to contexts where social distancing
might be needed (e.g., not for the Road Traffic library, and not generally for
transporters in the Material Handling Library):

  * Using the Pedestrian Library to model free-space movement in built
    environments (including realistic grouping behaviour) and interaction with
    common built-environment features such as service points/queues and
    escalators.

  * Using the Process Modeling Library to move either entity or resource agents
    via networks of nodes and paths.

  * Use of direct `moveTo` commands in purer agent-based models.

Only the first of these considers other agents (and space markup) as obstacles
by default, so is a natural candidate for also including social distancing. (It
is certainly possible to implement movement 'interaction rules' using the other
techniques, but this effectively requires the modeller to develop those
from-scratch; e.g., by splitting routes into segments and 'reserving' them
beforehand, or moving between segments with decision logic on when/how to
proceed on each new segment.)

Within the Pedestrian Library, movement is included within the PedGoTo block and
implicitly within other blocks (e.g., PedService, PedWait). There is no direct
way to change the 'rules' of movement used; whilst one can change the agents'
size (diameter) so that they are *forced* to stay a certain distance from each
other, that is unusable because it causes lots of other problems (e.g., that the
pedestrians can't physically get through gaps smaller than their artificial
diameter).

The model here shows one way to add social distancing behaviour in a custom
block which acts as an 'extended' PedGoTo block, retaining the same interface
and functionality from the end user's perspective (just with some social
distancing specifics on top). Since this is just a PedGoTo block replacement, it
does not handle similar requirements in other blocks, though it could
potentially form part of the internal functionality of other custom blocks which
did. As such, it is not a 'full library' for social distancing via the
Pedestrian Library, but it does show ways to move towards this.

## Custom Block Functionality Summary

The demo model and the property descriptions in the block itself cover a lot of
this, but we provide a more general summary here (referencing the block's
properties as we go).

The **PedDistancingGoTo** block acts exactly like a normal PedGoTo block but
with extra properties (in their own "Social distancing rules" section) that
govern the social-distancing-specific behaviour.

The enhanced behaviour is as follows:

  * The agent type being used for the pedestrian needs to be specified in the
    block (in the **Pedestrian agent type** property). This agent type needs to
    provide certain AnyLogic functions which provide the block with information
    it needs, and also allow the agent to receive information about the scanning
    which it can use for things like animation. These functions are as follows:

      - **getSocialDistanceMetres**: returns the social distance (in metres)
        this pedestrian is trying to maintain. Note that this allows pedestrians
        to have different social distances, and thus model 'mixed' scenarios
        where some people are less concerned about social distance than others.

      - **getNonLineObstaclesForCurrentLevel**: returns a list of all markup
        shapes which occupy an area and act as obstacles for the pedestrian on
        its current level. This is needed so that the custom block does not
        attempt to move pedestrians inside obstacles (e.g., circular walls).
        Normally, this can just be set up as a fixed AnyLogic collection (of
        type **ArrayList**), listing all relevant markup shapes per level, that
        the function returns.

      - **scanDistanceOutcomeCallback**: every time the pedestrian performs a
        scan inside the custom block, this function will be called providing the
        distance to the nearest other pedestrian (or **null** if there isn't
        one). This allows the pedestrian agent type to perform its own logic
        using this information, such as animating its social distancing status
        (which the demo model shows).

    Technically, the agent type is declaring that it implements the
    DistancingPedestrian interface, which therefore needs listing in the
    **Advanced Java --> Implements** section of its properties. Each of the
    functions also needs to have a public access level (an **Access** of public
    in each function's **Advanced** properties).

  * As each pedestrian moves towards its target (or along its route), it will
    scan for nearby pedestrians at the specified **scanning interval**. (Each
    pedestrian has independent scanning intervals based on when they entered the
    block, which means that pedestrians will not always react at the same time
    to being too close to each other; this also makes the movements a bit more
    realistic and less 'choreographed'.)

  * If the nearest pedestrian is within its desired social distance, the
    pedestrian will make a movement adjustment: it will move a fixed distance
    (**Movement adjustment** property) directly away from that pedestrian. To
    more gracefully handle situations where this might mean trying to walk
    around a wall or similar (and thus taking a very long route), this movement
    will stop after the time it would take to move there in a straight line at
    the pedestrian's current speed.

    Once its adjustment is complete, it will continue with its movement to its
    original destination after a short delay (**Post-adjustment delay**
    property).

  * The pedestrian has a maximum number of movement adjustments it will try to
    make on its way to its original destination (**Max movement adjustments**
    property). This avoids artificial 'infinite adjustments' situations.

  * The pedestrian will exit the custom block when it completes its original
    desired movement (as per a normal PedGoTo block).


