---
layout: post
title:  "Solar Simulation"
author: "Janne Koskinen, Eero Bragge, Lars Aikala, Petro Soininen"
author-link: "http://appelfish.cloudapp.net"
date:   2015-08-31 10:00:00
categories: IoT Azure Arduino Raspberry Pi agriculture plant seed LED light solar simulation greenhouse growth chamber
color: "blue"
excerpt: "IoT service and LED light based simulation of outdoor light in three dimensions: intensity, light spectrum and time."
---

# Summary

This paper will describe architecture choices (both hardware and software) and technical solutions used to create an IoT solution for agricultural research and solution development.  The system, called LightDNA – Dynamic Outdoor Light, provides a cloud backed lighting system  capable of precisely  creating outdoor lighting conditions in controlled plant growing environments, for example in growth chambers and green houses.

After an introduction to the problem space and opportunity in solar light simulation the reader will learn about how Microsoft Azure and the Nitrogen [[1]](http://nitrogen.io) IoT framework were configured to create a full-fledged IoT solution with Raspberry Pi, Arduino and custom sensor hardware. The architecture and implementation can also be applied to various other IoT scenarios and the material provides references to reusable code and tutorials to help design your custom service.

Microsoft and Valoya [[2]](http://www.valoya.com/), a Finnish company specializing in plant lighting for professional plant growers, used the technology described in this paper to deploy a solar light simulation pilot installation to Queen Mary University in London to assist in their climate research.

Read the Valoya [press release](http://www.valoya.com/media-press) about the product and pilot.

# The Problem

Seed and crop protection companies, as well as research institutes and universities are doing plant research all over the world. Research target is often to find optimal inputs and plant varieties for different growing environments in order to increase crop yields or quality. Crop yields are critical for feeding the world, as plants are a vital part of human nutrition, whether consumed as fresh plants, fruit, seeds or after being processed to e.g. bread. Plants are also important indirectly as they are the main nutrition of many animals which humans consume as meat or from which we get dairy products. Thus research in these areas is important and improvements can have a significant impact on making our future more sustainable.

Currently, research of local plant varieties is typically done at or close to the conditions where the plants would ultimately be used. The limitations from the environment are obvious; winter stops the research in some areas as summer crops do not grow then and in some cases advanced laboratories and research centers may have relevant know-how, but lack suitable access to relevant environmental conditions. These issues have been alleviated with the use of simulated growing environments in growth rooms and chambers.

Temperature, Humidity, CO2-levels, and even light intensity are easy to simulate in controlled non-natural light environments. However, simulating the different variations in outdoor light quality is very challenging. Outdoor light varies more or less constantly in terms of light intensity and light quality (light spectrum) as well as the ever changing duration of day light (towards longer in the summer solstice and then again shorter days towards winter solstice).

Simulating visible light (about 400-700 nm) and a bit beyond, about 380-800 nm, covers most of the relevant areas of light to plants. Simulating this light range, means that there must be correct amount of light of each nm of wavelength. The combination of each nanometer step, results in the shape of the light spectrum. This light spectrum is very different in early mornings and evenings (dominantly red) than at midday clear sky. (See 1).

Figure 1 Solar light spectrum at morning and midday at same location

![Solar light at morning]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image01.jpg)
![Solar light spectrum at morning]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image02.png)
![Solar light at midday]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image03.png)
![Solar light spectrum at midday]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image04.png)

To have location based freedom in accurate research, the researchers need a light system that can bring the relevant light conditions to their laboratory and controlled environment growth chambers.

# Overview of the Solution

Valoya and Microsoft designed a system where a LED fixture (or a group of fixtures) can accurately simulate outdoor light spectrum and intensity in any location in real-time or at accelerated speed (for example simulating the spectrum changes of a full day in 30 minutes).  The creation of this system, was enabled by Valoya's development of special LEDs with unique spectral properties, the advances in power electronics and the maturing cloud and embedded computing infrastructure.

# Hardware and Software Setup

The solution is comprised of a combination of hardware:

- LED light fixtures with 8 unique channels of independently controllable LEDs.
- Dimmable power units, 8 per LED fixture, each accessible via PWM control.
- Arduino computer, acting as an interface between the power units and the Raspberry Pi. The Arduino has 0-5 volt analog output ports to connect to the power units
- Raspberry Pi with USB connection to the Arduino, wireless/wired connectivity to the internet and a memory card for offline storage and control programs.
- Local sensors for monitoring system output and operating conditions (temperature, humidity, light spectrum, light intensity, etc.).

And software:

- Local software on the Raspberry Pi (Linux based distribution) and on the Arduino
- Cloud based software on the internet running in Azure

The core idea of the system is simple. A target light spectrum and intensity, the outdoor light at a given time, is mimicked accurately by the LED system, which consists of carefully selected LEDs with suitable light spectra. In practice the individually controlled dimming levels of 8 groups of LEDs in the fixtures, dimmed up or down in a way, so that the highest possible match to the target spectrum is achieved. This is done with a specific optimization algorithm.

Following figures 2 – 5 explain how precisely the sun's spectrum can be simulated. Figure 2 shows 8 uniformly distributed theoretical light sources each with a smooth Gaussian form.  Figure shows the optimal fit using these sources. The sun's spectrum has sharp drops due to atmospheric gas absorption. The biggest drop in figure 3 is caused by atmosphere's oxygen (O2). So without adding these gas absorption bands to the system somehow a 100% match cannot be achieved.

![Figure 2 Theoretical evenly distributed light sources]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image05.png)

![Figure 3 Optimal match using theoretical light sources]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image06.png)

The spectra of real life light sources are not as smooth and spectra have unwanted bumps as shown in figure 4. Figure 5 shows a typical optimal match using real life light sources. When the lighting system has enough different types of sources a 90 – 95% match can be achieved.

![Figure 4 Example spectrum of a real life light source]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image07.png)

![Figure 5 Optimal match using real life light sources]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image08.png)

The software communicates the dimming levels, through PWM to the dimmable power units, which then execute the dimming of the LED by chaining the current, which powers the LEDs and determines the light output. Mean Well LCM-series dimming settings can be controlled with PWM (Pulse Width Modulation) analog signals. PWM is also an output mode of the Arduino microcontrollers.  Raspberry Pi2 was selected as the master computer (2) providing communication between the Arduino, local sensors and the cloud component.

# Pilot Installation at Queen Mary University London

The pilot installation consists of one Raspberry Pi2 and one Arduino Mega controlling 72 Mean Well power units and 9 physical LED modules. The Arduino has 8 PWM output channels connected to 8 Mean Well drivers and these drivers together control 1 LED module consisting of 8 LED channels. The remaining 8 Mean Well drivers are chained with custom cables to control the other 8 LED modules. See figures below.

Figure 6, 9 LED modules having enough power to simulate solar light anywhere in world

![Pilot installation light unit]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image09.png)
![Pilot installation control unit]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image10.png)

![Figure 7, Pi, Arduino and a subset of the Mean Well drivers]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image11.png)

# Software Architecture

Valoya's goal is to offer the system as a service to a wide range of customers. Cloud based implementation of system monitoring and control is a natural fit for this goal and the pilot implementation sets a solid foundation on building a multi-tenant backend to serve customers with varying local light control, sensor data ingestion and data analytics requirements.

![Figure 8, Architecture of software solution]({{site.baseurl}}/images/2015-08-31-Solar-Simulation_images/image12.png)

Nitrogen.io IoT framework was selected to handle communications/messaging, authentication/authorization and device provisioning. The system administrator and user UI for managing lighting programs and monitoring system state was implemented on top of Oxide Web UI, which in turn sits mainly on top of Ember.js MVC framework. Oxide fits naturally with Nitrogen and provides a customizable skeleton for building IoT service web UIs.

# Configuring Nitrogen.io for IoT services

Nitrogen provides a reusable foundation for typical IoT scenarios and we did not have to extend or customize the framework to support the required use cases. While Nitrogen can be run as a hosted service, the dev team opted run a dedicated instance of Nitrogen on Azure – this provides flexibility for future customization and scalability.

The current pilot deployment of the solar light simulation system has low bandwidth requirements and a fairly relaxed uptime criteria, so the Nitrogen service and Oxide UI are running in one Linux virtual machine (Large Instance in Azure) This blog post walks you through how to deploy a similar configuration to Azure: "Setting up Nitrogen service manually" [[3]](http://appelfish.cloudapp.net/?p=39)

Preparing for the future, we also tested deploying the service to a scalable and fault tolerant environment. Read more about that from this post: "How to create Nitrogen cluster in Azure using Deis" [[4]](http://appelfish.cloudapp.net/?p=76).

# On-premise Software Setup

The Raspberry Pi manages communication to the cloud, the light controlling Arduinos and to optional locally available sensors.

Nitrogen's Node.js library provides an easy way to implement communication between the elements. For setting up communication to the Arduino device(s), the code enumerates Raspberry Pi's USB ports and creates own process for every Arduino and registers devices to listen for different Nitrogen commands like new light program command.  To see how this works at detailed level, we also posted a blog article how easy it is to control and monitor devices with Nitrogen [[5]](http://appelfish.cloudapp.net/?p=90).

Light program files are csv-files where each row is solar light spectrum for specific time. First column of file describes when that spectrum should be used and if time difference between rows is more than one minute, system will interpolate values between these rows once per minute.

# Telemetry Data Collection and Analytics

The outdoor light simulation system can be also extended to perform data ingestion from local or remote sensors, e.g. spectrometer, temperature, humidity and moisture sensors. With a local spectrometer, it is easy to follow how well the light produced by the LED modules matches the target spectrum. Normally this should be between 60% and 95% depending on conditions. The LED array can be further optimized to reach high match to target spectrum. In our development phase setup an Ocean Optics spectrometer was connected via USB to the Raspberry Pi. Temperature, humidity and moisture sensors were hooked up to the Arduino – which in turn was connected via USB to the Raspberry Pi.

All telemetry data is sent to the Nitrogen service once per second except spectrometer data that is sent only when light dimming levels are changed. Nitrogen can be configured to automatically relay the data to Azure Event Hubs for integration to an analytics pipeline.

# Reusable Assets

The system architecture can be reused in other IoT projects. "How to make super easy IoT applications" blog article [[5]](http://appelfish.cloudapp.net/?p=90) provides a jump start to get started implementing your own system.

When calculating optimal LED dimming levels we actually calculate what is the closest match to the target spectrum which can be theoretically reached by adding together individual spectral values from the system LEDs. 

We also open-sourced our Node.js implementation as an npm library [[9]](https://www.npmjs.com/package/linearleastsquarescurvefit) to enable reuse by other developers.

The web UI for system control and administration is built on top of a forked version of nitrogenjs/oxide. The fork is available GitHub [[7]](https://github.com/jtjk/oxide) for easy reuse in other IoT apps. Difference to the original version is that the fork uses EmberJs DS.RESTAdapter instead of DS.LSAdapter which provides automatic caching and ensures fresh data when needed.

# Links to Blogs and Code Repositories

1. Nitrogen IoT framework  [http://nitrogen.io](http://nitrogen.io)
2. Valoya website [http://www.valoya.com/](http://www.valoya.com/)
3. Setting up Nitrogen service manually - [http://appelfish.cloudapp.net/?p=39](http://appelfish.cloudapp.net/?p=39)
4. How to create Nitrogen cluster in Azure using Deis - [http://appelfish.cloudapp.net/?p=76](http://appelfish.cloudapp.net/?p=76)
5. How to make super easy IoT applications - [http://appelfish.cloudapp.net/?p=90](http://appelfish.cloudapp.net/?p=90)
6. 0-5V to 0-10V PWM signal converter for Arduino - [http://birota.azurewebsites.net/0-5v-to-0-10v-pwm-converter-for-arduino/](http://birota.azurewebsites.net/0-5v-to-0-10v-pwm-converter-for-arduino/)
7. Oxide UI RESTAdapter fork in GitHub - [https://github.com/jtjk/oxide](https://github.com/jtjk/oxide)
8. LinearLeastSquareCurveFit in GitHub - [https://github.com/ebragge/LinearLeastSquaresCurveFit](https://github.com/ebragge/LinearLeastSquaresCurveFit)
9. LinearLeastSquareCurveFit in NPM repository - [https://www.npmjs.com/package/linearleastsquarescurvefit](https://www.npmjs.com/package/linearleastsquarescurvefit)
