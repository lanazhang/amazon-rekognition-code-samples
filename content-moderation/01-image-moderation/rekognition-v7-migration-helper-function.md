# Amazon Rekognition Moderation V7 migration - a helper function

Below is a sample Python function showcasing how you can create a "backward compatible" function to convert V7 response JSON to the V6 format using the above mapping. 
> :warning: **Please note that:** this is not a recommended way to migrate your code. It serves as a quick solution to convert the code, providing more time for your technical team to migrate the taxonomy properly.

### Declare a data structure that maintains the mapping from the moderation V7 label names to V6 and previous version labels.
The dictionary is generated based on the L1/L2 mappings mentioned above. It serves as a lookup dataset, helping us quickly map the V7 response to V6 label names.
``` 
v7_name_mapping = {"Exposed Female Nipples": [{"Name": "Nudity", "ParentName": "Explicit Nudity"}], "Exposed Buttocks": [{"Name": "Nudity", "ParentName": "Explicit Nudity"}], "Exposed Male Genitalia": [{"Name": "Graphic Male Nudity", "ParentName": "Explicit Nudity"}], "Exposed Female Genitalia": [{"Name": "Graphic Female Nudity", "ParentName": "Explicit Nudity"}], "Explicit Sexual Activity": [{"Name": "Sexual Activity", "ParentName": "Explicit Nudity"}, {"Name": "Illustrated Explicit Nudity", "ParentName": "Explicit Nudity"}], "Explicit Nudity": [{"Name": "Illustrated Explicit Nudity", "ParentName": "Explicit Nudity"}], "Sex Toys": [{"Name": "Adult Toys", "ParentName": "Explicit Nudity"}], "Female Swimwear or Underwear": [{"Name": "Female Swimwear Or Underwear", "ParentName": "Suggestive"}], "Male Swimwear or Underwear": [{"Name": "Male Swimwear Or Underwear", "ParentName": "Suggestive"}], "Implied Nudity": [{"Name": "Partial Nudity", "ParentName": "Suggestive"}], "Exposed Male Nipple": [{"Name": "Barechested Male", "ParentName": "Suggestive"}], "Non-Explicit Nudity": [{"Name": "Revealing Clothes", "ParentName": "Suggestive"}], "Obstructed intimate parts": [{"Name": "Revealing Clothes", "ParentName": "Suggestive"}], "Kissing on the lips": [{"Name": "Sexual Situations", "ParentName": "Suggestive"}], "Blood & Gore": [{"Name": "Graphic Violence Or Gore", "ParentName": "Violence"}], "Physical Violence": [{"Name": "Physical Violence", "ParentName": "Violence"}], "Weapon Violence": [{"Name": "Weapon Violence", "ParentName": "Violence"}], "Weapons": [{"Name": "Weapons", "ParentName": "Violence"}], "Self-Harm": [{"Name": "Self Injury", "ParentName": "Violence"}], "Emaciated Bodies": [{"Name": "Emaciated Bodies", "ParentName": "Visually Disturbing"}], "Corpses": [{"Name": "Corpses", "ParentName": "Visually Disturbing"}, {"Name": "Hanging", "ParentName": "Visually Disturbing"}], "Air Crash": [{"Name": "Air Crash", "ParentName": "Visually Disturbing"}], "Explosions and Blasts": [{"Name": "Explosions And Blasts", "ParentName": "Visually Disturbing"}], "Middle Finger": [{"Name": "Middle Finger", "ParentName": "Rude Gestures"}], "Products": [{"Name": "Drug Products", "ParentName": "Drugs"}, {"Name": "Tobacco Products", "ParentName": "Tobacco"}], "Drug & Tobacco Paraphernalia & Use": [{"Name": "Drug Use", "ParentName": "Drugs"}, {"Name": "Drug Paraphernalia", "ParentName": "Drugs"}], "Pills": [{"Name": "Pills", "ParentName": "Drugs"}], "Smoking": [{"Name": "Smoking", "ParentName": "Tobacco"}], "Drinking": [{"Name": "Drinking", "ParentName": "Alcohol"}], "Alcoholic Beverages": [{"Name": "Alcoholic Beverages", "ParentName": "Alcohol"}], "Gambling": [{"Name": "Gambling", "ParentName": "Gambling"}], "Nazi Party": [{"Name": "Nazi Party", "ParentName": "Hate Symbols"}], "White Supremacy": [{"Name": "White Supremacy", "ParentName": "Hate Symbols"}], "Extremist": [{"Name": "Extremist", "ParentName": "Hate Symbols"}]}
```

### Create the `v7_to_v6` function to convert the provided V7 JSON response to the V6 label values.
This function will return the original request as it is if the model version is 6.1.
```python
def v7_to_v6(v7_response):
    # Validation and ignore v6 response
    if v7_response is None or "ModerationLabels" not in v7_response or v7_response.get("ModerationModelVersion") == "6.1":
        return v7_response
    
    result = {
        "ModerationLabels": [],
        "ModerationModelVersion": "7.0 mapped",
        "ResponseMetadata": v7_response.get("ResponseMetadata")
    }    
    
    # convert all the labels and set is_explicit=True if response contains "Explicit"
    is_explicit = False
    if "ModerationLabels" in v7_response:
        for item in v7_response["ModerationLabels"]:
            v6_items = v7_name_mapping.get(item["Name"])
            if v6_items is not None:
                for v6i in v6_items:
                    v6i["Confidence"] = item["Confidence"]
                    result["ModerationLabels"].append(v6i)
                    
                    if v6i["Name"] == "Explicit":
                        is_explicit = True

    # Return v6 "Illustrated Explicit Nudity" label if V7 return "Explicit" along with "ContentType" in "illustrated" or "animated"
    if is_explicit and "ContentTypes" in v7_response:
        for ct in v7_response["ContentTypes"]:
            if "Name" in ct and ct["Name"].lower() in ["illustrated", "animated"]:
                result["ModerationLabels"].append(
                    {
                        "Confidence": ct["Confidence"],
                        "Name": "Illustrated Explicit Nudity",
                        "ParentName": "Explicit Nudity",
                    }
                )
                break   
                                            
    return result
```
### Below is a sample JSON response from the Rekognition DetectModeration V7 API.
```python
v7_response = {
    "ModerationLabels": [
        {
            "Confidence": 99.44782257080078,
            "Name": "Smoking",
            "ParentName": "Drug & Tobacco Paraphernalia & Use",
            "TaxonomyLevel": 3
        },
        {
            "Confidence": 99.44782257080078,
            "Name": "Drug & Tobacco Paraphernalia & Use",
            "ParentName": "Drugs & Tobacco",
            "TaxonomyLevel": 2
        },
        {
            "Confidence": 99.44782257080078,
            "Name": "Drugs & Tobacco",
            "ParentName": "",
            "TaxonomyLevel": 1
        }
    ],
    "ModerationModelVersion": "7.0",
    "ContentTypes": [{"Name":"illustrated","Confidence":99.9999}]
}
```
### Sample usage involves invoking this function by providing the Rekognition V7 moderation response, and it will convert it back to the mapped V6 labels.
```python
v6_format_response = v7_to_v6(v7_response)
```

### The response returned from the conversion function is compatible with V6.1 and previous versions, as illustrated below.
```json
{
  "ModerationLabels": [
    {
      "Name": "Smoking",
      "ParentName": "Tobacco",
      "Confidence": 99.44782257080078
    },
    {
      "Name": "Drug Use",
      "ParentName": "Drugs",
      "Confidence": 99.44782257080078
    },
    {
      "Name": "Drug Paraphernalia",
      "ParentName": "Drugs",
      "Confidence": 99.44782257080078
    }
  ],
  "ModerationModelVersion": "7.0 mapped"
}
```

### Example that function with the Rekognition DetectModerationLabels API call:
```python h1_lines="3"
import boto3
rekognition = boto3.client('rekognition')
response = v7_to_v6(rekognition.detect_moderation_labels(
    Image={ 
          "S3Object": { 
             "Bucket": "S3_BUCKET",
             "Name": "S3_KEY"
          }
       }
))
```