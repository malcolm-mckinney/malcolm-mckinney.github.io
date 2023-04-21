---
title: "Over-The-Counter Class I Medicines" # Title of your project
date: 2023-04-21T19:24:58+09:00
weight: 1 # Order in which to show this project on the home page
external_link: "" # Optional external link instead of modal
resources:
    - src: plant.jpg
      params:
          weight: -100 # Optional weighting for a specific image in this project folder
---
Back in 2016, there were not many easy and convenient ways to order over-the-counter class one medicines online in Japan.
To explain briefly, class one is a special designation of drug, in which a pharmacist must approve the use of before sale.

The general flow is such: if one is experiencing symptoms of disease, he or she would first would go to a doctor to correctly diagnose the issue.
The doctor, after performing tests on the patient, would, in turn, present to the patient a script, which would be presented to a pharmacist.
Once the script has been handed over, the pharmacist would double-check the drugs and dosages and finally give the patient the required drugs.
Going through this process is, indeed, time-intensive and moreover, the patient can become quite distressed especially if the illness is a sensitive matter.
I firmly believe that doctors and pharmacists alike should never shame or berate their patients for their illnesses or situations, but sadly this is not always the case.
I am proud to have worked on a project that helps people to get the medical help that they need anonymously and online--and--without shame.

If you ponder a bit, an interesting question begins to emerge: how would you automate a manual process like this?

Legally, such a process can not be fully automated, precisely because a licensed pharmacist must be involved strictly before the drugs are distributed to the customer.
In order to satisfy the legal requirements, Amazon Japan had to hire several pharmacists full time just for this project.
What can be automated, however, is the creation and transmission of a patient's medical data for review by pharmacists.

My direct involvement in this project was the creation of the mobile and desktop questionnaire page in which the customer can fill out the relevant and required medical information for pharmacist review.

In order for the questionnaire to be successfully injected into the checkout pipeline, a new attribute had to be created for ASINs with the class one medicine designation.
During the checkout process, if the customer has any products with the OVER_THE_COUNTER_CLASS_ONE attribute set to "true", a hook is triggered and the customer is redirected to the medical questionnaire page.

As you can see in the video below, the medical questionnaire has front-end JavaScript validation to mitigate mistakes before submission.
Upon submission, although there are not the same kind of HIPPA-style laws in Japan as there are in the United States, it is necessary to comply with Amazon's customer data policy. Therefore, the customer's medical data was to be stored encrypted in a secured database.
As soon as the customer's medical data is submitted and becomes available in the database, the medical information can be seen by the pharmacists on their internal management system that our team created.
If there are no concerns with the information provided by the customer, the pharmacists can approve the order and the medicines will be shipped to the customer.

Just only a few days after we soft-launched this service, to our surprise, TechCrunch Japan reported on the new feature and questioned whether this is a sign that Amazon would become a huge player in online medicine space.
Given how our world has changed so much only a mere four years later, it is very likely that more automation in medicine will come in the near future. Whether Amazon will be the one to invest in lead this undertaking is a topic of much intrigue.
