---
layout: post
title:  "Integrating with a Smart Environment: Smart Fire Evacuation and Rescue"
date:   2020-07-18 17:47:27 +0800
categories: IoT AI
---
At this year's **[SCDF X IBM - Lifesavers' Innovation Challenge: Call For Code 2020](https://www.scdf.gov.sg/scdf_innovation_challenge/About)**, my team, Dr Watson, was amongst 370 teams challenged to come up with innovative solutions tackling SCDF’s most pressing concerns around crisis management and climate change responsiveness. We chose to focus on the problem statement "Integrating with a Smart Environment":

> Infrastructure is getting “smart”, with sensors and Internet of things (IoT) increasingly embedded in the built environment (e.g. Punggol Digital District). How might we leverage a network of smart infrastructure in the built environment to make better and more timely sense of emergency incidents (e.g. detection of fires developing, building collapses, falls, road traffic accidents etc.) and to trigger early intervention measures, without the need to activate precious emergency resources?

## Dissecting the Problem
Breaking down the problem statement, we first looked at the possible scenarios - fire outbreaks, building collapses, car accidents - scenarios which often result in mass casualties. The optimal outcome is one in which early intervention measures can successfully minimise damage done and require minimal activation of emergency resources. Some challenges we brainstormed were a lack of real-time information for responders during emergencies and victims not knowing what to do.
With smart city infrastructure, there is an opportunity to use sensors for real-time monitoring and data-driven insights.

We decided to tackle fire emergencies. In such situations, there is a need for continuous on-the-ground intelligence for effective decision-making so that evacuation and rescue can be safer for all parties. Our three key stakeholders are the civilians inside the building, building security, and the SCDF.

## Stakeholders
### People within the Building
Currently, there are a few possible scenarios that could put these people at risk of running to a chokepoint or dangerous zones during a panicky evacuation:
- Lack of familiarity with the building layout
- Lack of real-time information of blockages from the fire and alternative exit routes
- Stranding due to mobility constraints

### First Responders: Security, Management, Community First Responders
For this group of people, the priority would be to rush to the key areas that need help directing people to the nearest safest exit or areas where there is a need for assistance, such as people with disabilities. However, current CCTVs are not intelligent in tracking areas and people that need assistance, wasting security and management’s time for an effective on the ground evacuation efforts.

### SCDF First Responders
For the SCDF emergency team rushing to the incident site, they also face a series of challenges in planning and executing a quick and effective rescue plan:
- There is an information lag as they rely on the information provided by the caller/witnesses at the scene (i.e. Occupants at risk may have moved from their last known location)
- Insufficient real-time overview of the situation and damage to formulate a clear rescue plan
  - Lack of information of structural damages and blockages increase time to search for rescue routes
  - No clear latest situation of where the trapped people are

Having all these vital information for the different parties have great potential in significantly reducing the casualties in times of emergency like a fire breakout.

## Our Solution
Our team believes that with the rise of "Smart" Infrastructure, we can leverage existing and upcoming technologies to address these challenges. We have built a prototype evacuation and rescue system that makes use of smart intelligence collected from various data sources in the smart building to provide a clear, real-time situation on the ground through a 3-D building dashboard, highlighting the vital information for quick and effective decision-making by the first-responders and rescuers. The real-time intelligence also provides data to generate the safest and fastest evacuation routes for the people in the building.

To efficiently and safely evacuate occupants when a fire is detected, the system will make use of WiFi Triangulation to locate occupants and serve personalised optimal escape routes directly to their devices. The algorithm will tap on the data from sensors and CCTV cameras to determine which zones are unsafe/blocked and compute the best route to safety accordingly.

Upon the actuation of the fire alarm, the responders will have access to a dashboard with real-time updates of the situation at the site. The CCTV footage from the site will be processed to automatically identify (Computer Vision) zones that still contain a high number of occupants, as well as the zones where people with mobility impairments were detected (e.g. wheelchair bound occupants). Distress signals are also picked up by our sound sensors in the event that CCTVs' visibility is low. The dashboard will also contain a route finder that computes the fastest and safest way to move between any zone in the building, taking into account damage and blockages.

Our solution prototype could be implemented in existing or new buildings, as long as they have the infrastructure required (WiFi, CCTV Cameras and smoke/fire sensors).

Watch our pitch video here: http://www.youtube.com/watch?feature=player_embedded&v=FJVBh4wFeD4

Solution: http://dr-watson-to-the-rescue.s3-website-ap-southeast-1.amazonaws.com/

## Technologies Used
![](assets/scdf_img/c4c-arch.png)

It was a competition requirement that we use at least one IBM Cloud offering - we found 3 suitable for our design:
- **IBM Cloud Foundry** to serve our Flask web service (Python)
- **IBM Watson Visual Recognition** for detecting mobility aid devices and counting the number of stranded victims
- **IBM Developer Model Asset Exchange: Audio Classifier** for identifying sounds of distress

We served our React dashboard from **AWS Simple Storage Service**.

## Conclusion
I was really proud of what my team was able to come up with in just 48 hours. On reflection, I believe these are what enabled us to work so well together and even emerge champions:
- Critical thinking
- Complementary, diverse skill sets
- Initiative
- Frequent communication

I had a lot of fun and would not hesitate to work with these newfound friends again.