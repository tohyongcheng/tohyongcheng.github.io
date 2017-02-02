---
layout: post
title: Superset - the open source Tableau
comments: true
date:   2017-02-03 11:00:00
categories: "data", "data-engineering", "data-visualization"
---

## Introduction

I am just so amazed by AirBNB's Superset that I have decided to immediately write a blog post about this wonderful piece of work so that I can share some knowledge here.

After some time developing on data pipelines and workflows, one challenge we faced was how we can have greater flexibility with our data sets. One of the things I wanted to start at work was to have a data playground, where you can play with multiple sources of databases and make queries/aggregations quickly in dashboards just like Tableau.

Look no further, here comes Superset.

## What's Superset?

<img
  src="https://cloud.githubusercontent.com/assets/130878/20946612/49a8a25c-bbc0-11e6-8314-10bef902af51.png"
  alt="Superset"
  width="500"
/>

**Superset** is a data exploration platform designed to be visual, intuitive and interactive.

I call it an *instant database playground*.You can quickly create splices with out of the box charts, histograms or graphs. It started off as a hackathon project, and is now growing to be quite powerful as an open source solution.

AirBNB puts it very nicely:

> Superset allows data exploration through rich visualizations while performing fast and intuitive “slicing and dicing” against just about any dataset.

> Data explorers can easily travel through multi-dimensional datasets while creating and sharing “slices”, and assemble them in interactive dashboards.

I'll just get into the different pros and cons that I currently can think of it now:

#### Some magical GIFs from the repo:

**View Dashboards**
![superset-dashboard](https://cloud.githubusercontent.com/assets/130878/20371438/a703a2a0-ac19-11e6-80c4-00a47c2eb644.gif)

<br/>
**View/Edit a Slice**
![superset-explore-slice](https://cloud.githubusercontent.com/assets/130878/20372732/410392f4-ac22-11e6-9c6d-3ef512e81212.gif)

See more at https://github.com/airbnb/superset#screenshots--gifs

#### Here's a video of how it works:

<iframe width="560" height="315" src="https://www.youtube.com/embed/3Txm_nj_R7M" frameborder="0" allowfullscreen></iframe>


### Pros

- It's a free and good enough alternative to Tableau (which is ridiculously expensive!). 
- It can connect to many different types of SQL databases, through Python's SQLAlchemy
- Out of the box visualisations, and I'm sure many others will add more useful visualisations through open source
- Out of the box authentication and authorization system, including support for LDAP, OpenID, OAuth etc.
- Increasing support for Druid databases (which provides for quick OLAP queries for Business Intelligence)
- Setup time of 10 minutes!
- Easy to use for non-coders after it is setup!
- Built on Python, so it's easy to understand the code or even to start contributing.
- It comes with SQL Lab, where you can run SQL visualize queries on your browser


### Cons
- Very little support for now. Since it is an open solution, you can't expect more... But I'm sure the community will try to help
- Still little support for scalable architecture, but it could just be as easy as deploying a cluster of Superset Docker containers.
- No support for NoSQL databases as of yet, but I think it'll come soon: https://github.com/airbnb/superset/issues/600
- Not as fully featured as Tableau.
- Visualisations are still quite basic, but are good enough as a V1 prototype.

Obviously, the pros outweigh the cons if you ask me.

## What I did:

What I just was to follow the tutorial listed on the documentation page at http://airbnb.io/superset/installation.html where it comes with pre-installed examples to let you know how data can be visualised with its current set of visualisations. 

To be honest, I was quite impressed with what it can do. I just connected a Redshift cluster with some of the data that we already use for analytics and voila! I spent 30 minutes setting up the database, and also creating 3 'splices' and graphs of what I usually query for my work. Everything was saved on a dashboard, and I could immediately show to my colleagues. 

With Superset, your data evolves into a playground instantly. And it comes for free.

## How to setup:

In order to setup, follow the instructions here: http://airbnb.io/superset/installation.html

An overview of what you need to do is:

1. Install the python dependencies
2. Install superset through pip
3. Setup superset in the command line.
4. Load some examples
5. Setup administrator
6. Start server

At this point of time, you can start exploring the pre-loaded examples and see what visualisations it is capable of.

If you have a database of some data, you can immediately connect it to Superset by:
7. Connecting to database through SQLAlchemy
8. Setup tables that you think are relevant
9. Setup what you want to do with each column of the table, e.g whether it is filterable, groupable, to count as distinct etc
10. Create a splice of your data through a visualisation

It's that easy!

## Conclusion

If you get to see this, try it. It is quite amazing with that AirBNB has open-sourced such a useful solution for anyone to do quick visualisations with the data they have in hand. This will be useful for startups who don't want to spend much on a commercial solution, but still want to get some flexibility and quick prototyping capabilities with data visualisations. I think you can even bring it to a business meeting/presentation and show some immediate hands on skills to impress the viewers.

## References:
1. https://medium.com/airbnb-engineering/caravel-airbnb-s-data-exploration-platform-15a72aa610e5#.9ag81n6bt
2. http://airbnb.io/superset/
