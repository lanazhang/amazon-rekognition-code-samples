# Amazon Rekognition Moderation V7 migration instruction

On January 18 (TBD), 2024, Amazon Rekognition will release an updated machine learning model for moderation label detection, upgrading from version 6.1 to 7.0. This update will:
 
- Introduce new label categories 
-  Modify existing label categories
- Improve overall model accuracy
- Add capability to identify animated or illustrated content

To ensure a smooth transition, your account will continue using version 6.1 until February 19, 2024. During this time, you have a couple of options to test the new version 7.0 model:
- Use the private AWS SDK for Amazon Rekognition. This allows you to specify version 7.0 in your API requests. We will provide the download instructions for the SDK and documentation on January 15. 
- Upload images to the Amazon Rekognition Bulk Analysis Console or Content Moderation Demo Console. This will analyze images using version 7.0.

## Migrate your code from V6.1 to V7

Some customers employ post-processing logic following the DetectModerationLabels or StartContentModeration APIs call to conditionally include or exclude certain categories. Since some of the V7 label names differ from the previous version (V6.1), you'll need to revisit the code and make corresponding changes. 

We recommend existing customers using Rekognition DetectModerationLabels or StartContentModeration APIs review the new taxtonomy throughly and refactor your application. As there are new categories add and some existing category deinfition updated as well. For a full V7 taxtonomy, refer to this [document](https://docs.aws.amazon.com/rekognition/latest/dg/moderation.html#moderation-api).

## L1 Mapping
If you have post-processing logic that filters only on the top-level category (L1), such as `Explicit Nudity`, `Suggestive`, `Violence` etc., you can refer to the table below to update the code.
V6.1 L1 | V7 L1
---------- | ---------- 
Explicit Nudity | Explicit
Suggestive | Non-Explicit Nudity of Intimate parts and Kissing
&nbsp;| Swimwear or Underwear
Violence | Violence
Visually Disturbing | Visually Disturbing
Rude Gestures | Rude Gestures
Drugs | Drugs & Tobacco
Tobacco | Drugs & Tobacco
Alcohol | Alcohol
Gambling | Gambling
Hate Symbols | Hate Symbols

## L2 Mapping
If you have post-processing logic that filters on the second-level category (L2) as well, such as `Explicit Nudity / Nudity`, `Suggestive / Female Swimwear Or Underwear`, `Violence / Weapon Violence` etc., you can refer to the table below to update the code.
V6.1 L1 | V6.1 L2 | V7 L1 | V7 L2 | V7 L3 | V7 ContentTypes
---------- | ---------- | ---------- | ---------- | ---------- | ----------  
Explicit Nudity | Nudity | Explicit | Explicit Nudity | Exposed Female Nipples, Exposed Buttocks | 
&nbsp;| Graphic Male Nudity | Explicit | Explicit Nudity | Exposed Male Genitalia | 
&nbsp;| Graphic Female Nudity | Explicit | Explicit Nudity | Exposed Female Genitalia | 
&nbsp;| Sexual Activity | Explicit | Explicit Sexual Activity |&nbsp;|  
&nbsp;| Illustrated Explicit Nudity | Explicit | Explicit Nudity |&nbsp;|  Map to "animated" and "illustrated"
&nbsp;| Illustrated Explicit Nudity | Explicit | Explicit Sexual Activity |&nbsp;|  Map to "animated" and "illustrated"
&nbsp;| Adult Toys | Explicit | Sex Toys |&nbsp;|  
Suggestive | Female Swimwear Or Underwear | Swimwear or Underwear | Female Swimwear or Underwear |&nbsp;|  
&nbsp;| Male Swimwear Or Underwear | Swimwear or Underwear | Male Swimwear or Underwear |&nbsp;|  
&nbsp;| Partial Nudity | Non-Explicit Nudity of Intimate parts and Kissing | Non-Explicit Nudity | Implied Nudity | 
&nbsp;| Barechested Male | Non-Explicit Nudity of Intimate parts and Kissing | Non-Explicit Nudity | Exposed Male Nipple | 
&nbsp;| Revealing Clothes | Non-Explicit Nudity of Intimate parts and Kissing | Non-Explicit Nudity |&nbsp;|  
&nbsp;|&nbsp;| Non-Explicit Nudity of Intimate parts and Kissing | Obstructed intimate parts |&nbsp;|  
&nbsp;| Sexual Situations | Non-Explicit Nudity of Intimate parts and Kissing | Kissing on the lips |&nbsp;|  
Violence | Graphic Violence Or Gore | Violence | Graphic Violence | Blood & Gore | 
&nbsp; | Physical Violence | Violence | Graphic Violence | Physical Violence | 
&nbsp; | Weapon Violence | Violence | Graphic Violence | Weapon Violence | 
&nbsp; | Weapons | Violence | Weapons |&nbsp;|  
&nbsp; | Self Injury | Violence | Graphic Violence | Self-Harm | 
Visually Disturbing | Emaciated Bodies | Visually Disturbing | Death and Emaciation | Emaciated Bodies | 
&nbsp; | Corpses | Visually Disturbing | Death and Emaciation | Corpses | 
&nbsp; | Hanging | Violence | Violence | Corpses | 
&nbsp; | Air Crash | Visually Disturbing | Crashes | Air Crash | 
&nbsp; | Explosions And Blasts | Violence | Graphic Violence | Explosions and Blasts | 
Rude Gestures | Middle Finger | Rude Gestures | Middle Finger |&nbsp;|  
Drugs | Drug Products | Drugs & Tobacco | Products | &nbsp; | 
&nbsp; | Drug Use | Drugs & Tobacco | Drug & Tobacco Paraphernalia & Use |&nbsp;|  
&nbsp; | Pills | Drugs & Tobacco | Products | Pills | 
&nbsp; | Drug Paraphernalia | Drugs & Tobacco | Drug & Tobacco Paraphernalia & Use |&nbsp;|  
Tobacco | Tobacco Products | Drugs & Tobacco | Products |&nbsp;|  
&nbsp; | Smoking | Drugs & Tobacco | Drug & Tobacco Paraphernalia & Use | Smoking | 
Alcohol | Drinking | Alcohol | Alcohol Use | Drinking | 
&nbsp; | Alcoholic Beverages | Alcohol | Alcoholic Beverages |&nbsp;|  
Gambling | Gambling | Gambling | Gambling |&nbsp;|  
Hate Symbols | Nazi Party | Hate Symbols | Nazi Party |&nbsp;|  
&nbsp; | White Supremacy | Hate Symbols | White Supremacy |&nbsp;|  
&nbsp; | Extremist | Hate Symbols | Extremist |&nbsp;|  

### A helper function for a quick migration
You can find sample Python code for a quick helper function to assist you in converting the Rekognition moderation V7 response to the V6 format.
> :warning: **Please note that:** this is not a recommended way to migrate your code. It serves as a quick solution to convert the code, providing more time for your technical team to migrate the taxonomy properly.
[Helper function](rekognition-v7-migration-helper-function.md)