# Overview
Google Assistant is a personal voice assistant that offers a host of actions and integrations. From sending texts and setting reminders, to ordering coffee and playing your favorite songs, the 1 million+ actions available suit a wide range of voice command needs. Google Assistant is offered on Android and iOS, but it can even be integrated with other devices like smartwatches, Google Homes, and Android TVs.

As you will soon find out, Actions is the central platform for developing Google Assistant applications. Actions work with a number of human-computer interaction suites, which simplifies conversational app development. Out of all the platforms, the most popular is Dialogflow, which uses an underlying machine learning (ML) and natural language understanding (NLU) schema to build rich Assistant applications.

In this lab you will get hands-on practice with Actions and Dialogflow by building an Assistant application that generates quotes when prompted by a user. You will gain practical knowledge of computer-human interaction suites and by the end of this lab, you will have successfully built a fully-fledged Google Assistant application.

# What you will Learn
In this lab, you will learn how to:

* Differentiate between the basic components and services that make up an Assistant application.
* Create an Actions project.
* Integrate Dialogflow with an Actions project.
* Build a Dialogflow intent that handles quotation responses.
* Update your Google permission settings.
* Test your application with the Actions simulator.

# Prerequisites
This is an introductory level lab. This assumes little to no prior experience developing Google Assistant applications. However, basic knowledge of the Cloud Console and the Qwiklabs platform is expected. If you are a new Google Cloud or Qwiklabs user, please take the following lab before attempting this one:

A Tour of Qwiklabs and the Google Cloud
Once you're ready, scroll down to get your Actions project set up.

# Create an Actions project
Before we start building our quotation generator, let's review the following terms one more time.

Google Assistant is the virtual assistant that's found on smartphones, homes, and a host of other devices. It's the application that takes in voice commands and completes tasks based on user input.

Actions on Google is the developer platform that allows you to build applications for Google Assistant. This is going to be the central console for conversational application development.

Let's take our first step by building an Actions project.

Open a new tab in your browser and go to the Actions on Google Developer Console. Then sign in with your temporary lab credentials. Once you are signed in, you should be looking at a clean Actions console, which should resemble the following:

![action](assests/action1.png)

Click New Project and agree to the Actions on Google Terms of Service that appears. Next, click on the project field, select the Google Cloud project ID for Qwiklabs, and click IMPORT PROJECT:

![action](project-import.gif)

Soon after you will be presented with a welcome page that resembles the following:

![action](actions-console.png)

Click on Actions Console in the top left corner. Then click on the project you just created (title has your Project ID as the name.)