Welcome to the MoMoOpenAPI_SandBox wiki!

[](https://github.com/Ddamula/MoMoOpenAPI_SandBox/wiki#momo-open-api-sandbox-notebook)
[](https://github.com/Ddamula/MoMoOpenAPI_SandBox/wiki#authentication)
[](https://github.com/Ddamula/MoMoOpenAPI_SandBox/wiki#get-paid)
[](https://github.com/Ddamula/MoMoOpenAPI_SandBox/wiki#know-your-customer)
[](https://github.com/Ddamula/MoMoOpenAPI_SandBox/wiki#pay)
...

# <b>MoMo Open API SandBox NoteBook
...
```python 
from datetime import datetime
global Api_User 
global Api_Key 
GetPaid_Debit_Request_Ref_ID = "" #UUID String Request Reference 
Pay_Transfer_Request_Ref_ID = "" #UUID String Request Reference
Token=""
Token_expiry_time = ""
Token_expired = False
Token_expiry_time = datetime.now()
Environment = "sandbox" #Target Environment  
Collection_Subscription_Primary_Key  = "4c91dae7a6f1474387a23a1f3d448eb7"#Primary Key for Collection Subscription.https://momodeveloper.mtn.com/profile
Disbursement_Subscription_Primary_Key  = "dec90f29f4e14137912bfc3236d51cbe"#Primary Key for Disbursement Subscription.https://momodeveloper.mtn.com/profile
Base_Url = "https://sandbox.momodeveloper.mtn.com" #SandBox Base URL
```
# timer-wait-time
...
```python 
import time

def countdown(seconds):
    while seconds >= 1:
        print(str(seconds), end= ':')
        time.sleep(1)
        seconds -= 1
    print("0")
```
# <b>Authentication 
## Creating API User on the SandBox
...
```python 
#Function to create an API User (Username) from The MoMo OpenAPI SandBox
def Create_API_User_SandBox():
    import requests as rq
    import uuid
    global Api_User
    global Collection_Subscription_Primary_Key
    Api_User = str(uuid.uuid4())
    url = Base_Url+"/v1_0/apiuser"
    headers = {
    "X-Reference-Id": Api_User, #When creating Api user, the X Reference Id in the header will be created as the user. 
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Content-Type": "application/json"
    }
    body = {    
    "providerCallbackHost": "webhook.site" # If your callback is https://webhook.site/mycallback/site then the providerCallbackHost is webhook.site
    }
    try:
        resp = rq.request("post", url, json=body, headers=headers)
        if(str(resp.status_code)=="201"):
            print("HTTP Status Code:"+str(resp.status_code)+"\n Api user Created: "+Api_User)
        elif(str(resp.status_code)=="401"):
            print(str(resp.status_code)+" "+resp.text+" ")
            print("Ensure the subscription key is the primary")
        elif(str(resp.status_code)=="400"):
            print(str(resp.status_code)+" "+resp.text+" ")
            print("Ensure API User(X-Reference-Id) in the Headers is UUID Version 4")
            print("Ensure the Body contains the correct syntax ""\"providerCallbackHost""\""+":"+"Your CallBack URL HOST Eg ""\"webhook.site""\"")
        else:
            print(str(resp.status_code)+" "+resp.text+" ")
    except TypeError:
        print("Body of the Request has to be Json Format")
    except:
        print("Something Is Wrong "+resp.json)
#Create_API_User_SandBox()        
```
## Creating the API Key (Password) For the API USER Created SandBox
...
```python 
#Function to create an API Key for the API User from The MoMo OpenAPI SandBox
def Create_API_Key_SandBox():
    import requests as rq
    import traceback
    global Api_Key
    global Collection_Subscription_Primary_Key
    url = Base_Url+"/v1_0/apiuser/"+Api_User+"/apikey"
    headers = {
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    }    
    try:
        resp = rq.request("post", url, headers=headers)
        if(str(resp.status_code)=="201"):
            Response = resp.json()
            Api_Key = Response.get('apiKey')#Save the API Key in Variable 
            print("HTTP Status Code:"+str(resp.status_code)+"\n Api User:"+str(Api_User) +" Api Key:"+str(Api_Key))
        elif(str(resp.status_code)=="400"):
            print(str(resp.status_code)+" "+resp.text+" Validate the BaseURL \n And Ensure API_User is created, by calling the function Create_API_User_SandBox()")
        elif(str(resp.status_code)=="404"):
            print(str(resp.status_code)+" "+resp.text+" API_USER was not created, Please Run function Create_API_User_SandBox()")
        else:
            print(str(resp.status_code)+" "+resp.text+" ")
    except TypeError:
        print("Body of the Request has to be Json Format or No Body")
    except:
        print("Something Is Wrong ")
        traceback.print_exc()      
#Create_API_Key_SandBox()
```
### Test Functions Creating API User and Key
...
```python 
Create_API_User_SandBox() #Function create to create API User and store value in Variable {{Api_User}}
Create_API_Key_SandBox() #Function to create API Key(Password) for the Api User Created and stored in variable {{Api_Key}}

```

## Generate Access Token Function
To prevent the creation of tokens for each request, it is advisable to verify the token's validity and only generate a new one when the previous token has expired
...
```python 
#Function to generate Token and set Token expiry. 
def Get_Token():# function to return token (renews token if expired)
    import requests as rq
    import traceback
    import json
    from datetime import datetime, timedelta
    global Token 
    global Token_expiry_time
    EndPoint = Base_Url+"/collection/token/"
    Auth = bytes(Api_User + ':' + Api_Key, "utf-8")
    headers = {    
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    }
    try:
        resp = rq.request("post", EndPoint,auth=(Api_User,Api_Key), headers=headers)
        Response = resp.json()
        if(str(resp.status_code) == "200"):
            Token = Response.get('access_token')
            Token_expiry = Response.get('expires_in')
            Token_expiry_time = datetime.now() + timedelta(seconds= int(Token_expiry)) #Track Token Expiry Time 
            print("New Token Generated Expiring at :" +str(Token_expiry_time))           
        elif(str(resp.status_code) == "500" or str(Response.get("error"))=="login_failed"):
            print(Response)
            print("Ensure to Map the API User and API Key as (Username:Password) respectively")
        else:
            print(resp.text)            
    except:
        print("Something Is Wrong ")
        traceback.print_exc()  
#Get_Token()
```
## Validate  Access Token Status Function
....
```python 
#Function to Validate  Status of Token
#If the Token is Expired a new one will be generated. 
def Token_Status():
    if Token_expiry_time >= datetime.now():
        Token_expired = False    
        #print ("Token not Expired: Expiring at "+ str(Token_expiry_time))
        #print(Token)
    else:
        Token_expired = True
        Get_Token()
        #print ("New Token Generated Expiring at "+ str(Token_expiry_time))
        #print(Token)
##Token_Status()
```
# <b>Get Paid
## Get Paid Debit API Function
...
```python 
#Function that initiates a Debit USSD Prompt to the Payer to approve wit PIN
def Request_Debit_Payment(MSISDN,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global GetPaid_Debit_Request_Ref_ID
  Token_Status()
  GetPaid_Debit_Request_Ref_ID = str(uuid.uuid4())
  url = Base_Url+"/collection/v1_0/requesttopay"
  headers = {
    "X-Reference-Id": GetPaid_Debit_Request_Ref_ID, #Unique for every request, used to validate status of the request. 
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token, #Avoid creating new tokens for every request,  track the Expiry 
    "Content-Type": "application/json",
    "X-Callback-Url":"https://webhook.site/mycallback/site"### You can add X-Callback-Url to receive the callback  ("X-Callback-Url":"https://webhook.com/mysite/status")
  }
  body = {    
    "amount": Amount,
    "currency": "EUR", #use the currency as EUR in the SandBox
    "externalId": str(uuid.uuid1()), #Used for Reconciliation between application and MoMo platform. 
    "payer": {
      "partyIdType": "MSISDN",#EMAIL and ALIAS apply as well 
      "partyId": MSISDN
  },
    "payerMessage": "MoMo Debit API", #Message sent to the Payer
    "payeeNote": "MoMo Debit API" #Message Note to the  Payee
  }
  try:
    resp = rq.request("post", url, json=body, headers=headers)
    if(str(resp.status_code) == "202"):
      print("Debit request to MSISDN "+MSISDN+" Amount "+Amount+" "+ "Response Code "+str(resp.status_code))
      countdown(6)
      print("Request_Reference_ID :"+GetPaid_Debit_Request_Ref_ID )
    elif (str(resp.status_code) == "404"):
      print("Check The Base_URL ")
    elif (str(resp.status_code) == "400"):
      print("Ensure no Special Charters like & in the Message and Notes \nThe X-Reference-Id in the header should be UUID Versio 4")
      print(resp.text)
    elif (str(resp.status_code) == "500" or str(resp.json().get("message")).endswith("INVALID_CALLBACK_URL_HOST") or str(resp.json().get("message")).endswith("Currency not supported.")):
      print(resp.json())
      print("Ensure the  URL Host is the same with the one created when generating API_USer function ")
      print("Verify and validate  Currency for Sand Box is EUR")
    elif (str(resp.status_code) == "500" ):
      print(resp.text)
      print("API is not available")
    else:
      print(resp.status_code)
      print(resp.text)
  except TypeError:
    print("Request Body should be Json Formatted")
  except:
    print("Something Is Wrong ")
    traceback.print_exc()   
#Request_Debit_Payment("56733123453","50000")
```

## Check Debit Status Function 
...
```python 
#Check Status 
def Check_Debit_Status(X_Reference_Id_Of_The_Debit_Request):
    import requests as rq
    import json
    import traceback
    Token_Status()
    global Debit_Status
    url = Base_Url+"/collection/v1_0/requesttopay/"+X_Reference_Id_Of_The_Debit_Request
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        Status_Json_DD = str(Status_Json).replace('\'', '"')
        Debit_Status = Status_Json.get('status')
        print(str(Status_Json))
        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
    
#Check_Status("")

```

## Notification
Sending a Notification to the Payer after a successful Debit Request
...
```python 
#Send a Notification After a succesfull Request to Pay
def Send_Notification (X_Reference_Id_Of_The_Debit_Request):
    import requests as rq
    import json
    import traceback
    Token_Status()
    url = Base_Url+"/collection/v1_0/requesttopay/"+X_Reference_Id_Of_The_Debit_Request+"/deliverynotification"
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    body = {"notificationMessage": "Product has been Successfull sent to your Home, Thank You fro Purchasing from MoMo"}
    
    try:
        resp = rq.request("post", url, json=body, headers=headers)
        print(str(resp.status_code)+":Ok "+" Notification Sent")       
        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
```

### Testing Status Responses with payer_Id_msisdn
...
|  payer_id-Msisdn | Response status 
| --- | --- | 
46733123450	|Failed
46733123451	|Rejected
46733123452	|Timeout
56733123453	|Success
46733123454	|Pending
```python 
Request_Debit_Payment("56733123452","50000")
status = Check_Debit_Status(GetPaid_Debit_Request_Ref_ID)
if (Debit_Status == "SUCCESSFUL"):
    {
    Send_Notification(GetPaid_Debit_Request_Ref_ID)
}
else:
    print(Debit_Status+""+" Notification No Sent")
```
# <b>Know Your Customer
## Get Basic Info KYC Function
...
```python 
#Get Basic Info KYC (FirstName and LastName)
def Check_BasicInfo_KYC(Number,NAME):
    import requests as rq
    import json
    import traceback
    global First_Name, Last_Name,Proceed
    Token_Status()
    url = Base_Url+"/disbursement/v1_0/accountholder/msisdn/"+Number+"/basicuserinfo"
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        Status_Json_DD = str(Status_Json).replace('\'', '"')
        First_Name = Status_Json.get("given_name")
        Last_Name = Status_Json.get("family_name")
        if((str(First_Name).upper()+" "+str(Last_Name).upper()) ==NAME):
            Proceed = True
        else:
            Proceed = False        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
#Check_BasicInfo_KYC("56733123453","BOX SAND")

```

## Get Detailed KYC Function With Consent
...
```python 

```

### Generate Consent bc-authorize Function
...
```python 
#Send Consent, auth_req_id is generated as consent Id
def bc_authorize ():
    import requests as rq
    import json
    import traceback
    Token_Status()
    global Consent_ID
    url = Base_Url+"/disbursement/v1_0/bc-authorize"
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    'Content-Type': 'application/x-www-form-urlencoded'
    }
    body = 'scope=all_info&login_hint=ID:563667/MSISDN&access_type=offline'    
    try:
        resp = rq.request("post", url, data=body, headers=headers)
        resp_json = resp.json()
        Consent_ID =resp_json.get("auth_req_id")
        print(Consent_ID )
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
bc_authorize ()
```

### Generate Consent Token oauth2 Function
...
```python 
#Function to generate Oauth Token and set Token expiry. 
def Get_Oauth_Token(consent_auth_req_id):# function to return token (renews token if expired)
    import requests as rq
    import traceback
    import json
    from datetime import datetime, timedelta
    global Token_Oauth
    global Disbursement_Subscription_Primary_Key
    global Token_Oauth_expiry_time
    EndPoint = Base_Url+"/disbursement/oauth2/token/"
    #Auth = bytes(Api_User + ':' + Api_Key, "utf-8")
    headers = {    
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    'Content-Type': 'application/x-www-form-urlencoded',
    "X-Target-Environment": Environment
    }
    body = "grant_type=urn:openid:params:grant-type:ciba&auth_req_id="+consent_auth_req_id+""
    try:
        resp = rq.request("POST", EndPoint,auth=(Api_User,Api_Key), headers=headers,data=body)
        Response = resp.json()        
        if(str(resp.status_code) == "200"):
            #print(Response)
            Token_Oauth = Response.get('access_token')
            Token_Oauth_expiry = Response.get('expires_in')
            Token_Oauth_expiry_time = datetime.now() + timedelta(seconds= int(Token_Oauth_expiry)) #Track Token Expiry Time 
            print("New Token Generated Expiring at :" +str(Token_Oauth_expiry_time))           
        elif(str(resp.status_code) == "500" or str(Response.get("error"))=="login_failed"):
            print(Response)
            print("Ensure to Map the API User and API Key as (Username:Password) respectively")
        else:
            print(resp.text+" "+str(resp.status_code))
            print(rq)
    except:
        print("Something Is Wrong ")
        traceback.print_exc()  
#Get_Oauth_Token(Consent_ID) 

```

### Status Token oauth2 Function
...
```python 
#Function to Validate  Status of Token
#If the Token is Expired a new one will be generated. 
def Token_Oauth_Status():
    if Token_Oauth_expiry_time >= datetime.now():
        Token_expired = False    
        #print ("Token not Expired: Expiring at "+ str(Token_expiry_time))
        #print(Token)
    else:
        Token_expired = True
        Get_Oauth_Token(Consent_ID)
        #print ("New Token Generated Expiring at "+ str(Token_expiry_time))
        #print(Token)
#Token_Oauth_Status()
```

### Get Detailed KYC with Consent Token oauth2 Function
...
```python 
#Get Basic Info KYC (FirstName and LastName)
def Get_DetailedInfo_KYC():
    import requests as rq
    import json
    import traceback
    Token_Oauth_Status()
    url = Base_Url+"/disbursement/oauth2/v1_0/userinfo"
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token_Oauth
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Response = resp.json()
        print(resp.text)       
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
Get_DetailedInfo_KYC()
```
# <b>Pay
## Pay Disbursement Transfer Function
...
```python 
#Disbursements Transfer
#Function that initiates a Transfer from  Business  Wallet to a customer's  
def Request_Tranfer_Payment(MSISDN,MSISDN_NAME,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global Pay_Transfer_Request_Ref_ID 
  Token_Status()
  Check_BasicInfo_KYC(MSISDN,MSISDN_NAME)
  if(Proceed==True):
    Pay_Transfer_Request_Ref_ID  = str(uuid.uuid4())  
    url = Base_Url+"/disbursement/v1_0/transfer"
    headers = {
      "X-Reference-Id": Pay_Transfer_Request_Ref_ID , #Unique for every request, used to validate status of the request. 
      "X-Target-Environment": Environment,
      "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key ,
      "Authorization":"Bearer "+Token, #Avoid creating new tokens for every request,  track the Expiry 
      "Content-Type": "application/json",
      "X-Callback-Url":"https://webhook.site/mycallback/site"### You can add X-Callback-Url to receive the callback  ("X-Callback-Url":"https://webhook.com/mysite/status")
    }
    body = {    
      "amount": Amount,
      "currency": "EUR", #use the currency as EUR in the SandBox
      "externalId": str(uuid.uuid1()), #Used for Reconciliation between application and MoMo platform. 
      "payee": {
      "partyIdType": "MSISDN",#EMAIL and ALIAS apply as well 
      "partyId": MSISDN
    },
      "payerMessage": "MoMo Debit API", #Message sent to the Payer
      "payeeNote": "MoMo Debit API" #Message Note to the  Payee
    }
    try:
      resp = rq.request("post", url, json=body, headers=headers)
      if(str(resp.status_code) == "202"):
        print("Transfer request "+Pay_Transfer_Request_Ref_ID+" to MSISDN "+MSISDN+" Amount "+Amount+" "+ "Response Code "+str(resp.status_code))
        countdown(6)
        Check_Status_Transfer(Pay_Transfer_Request_Ref_ID)
        print("Completed")        
      elif (str(resp.status_code) == "404"):
        print("Check The Base_URL ")
      elif (str(resp.status_code) == "400"):
        print("Ensure no Special Charters like & in the Message and Notes \nThe X-Reference-Id in the header should be UUID Versio 4")
        print(resp.text)
      elif (str(resp.status_code) == "500" or str(resp.json().get("message")).endswith("INVALID_CALLBACK_URL_HOST") or str(resp.json().get("message")).endswith("Currency not supported.")):
        print(resp.json())
        print("Ensure the  URL Host is the same with the one created when generating API_USer function ")
        print("Verify and validate  Currency for Sand Box is EUR")
      elif (str(resp.status_code) == "500" ):
        print(resp.text)
        print("API is not available")
      else:
        print(resp.status_code)
        print(resp.text)
    except TypeError:
      print("Request Body should be Json Formatted")
    except:
      print("Something Is Wrong ")
      traceback.print_exc()   
  else:
    print("Names do not match")
#Request_Tranfer_Payment("56733123453","50000")
```

## Get Status on Disbursements Transfer Function
...
```python 
#Check Status Disbursements Transfer
def Check_Status_Transfer(X_Reference_Id_Of_The_Transfer_Request):
    import requests as rq
    import json
    import traceback
    Token_Status()
    url = Base_Url+"/disbursement/v1_0/transfer/"+X_Reference_Id_Of_The_Transfer_Request
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        Status_Json_DD = str(Status_Json).replace('\'', '"')
        print(Status_Json)        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
    
#Check_Status_Transfer("Check_Status_Transfer")

```
### Testing Status Responses with payee_Id_msisdn
|  payee_id-Msisdn | Response status 
| --- | --- | 
46733123450	|Failed
46733123451	|Rejected
46733123452	|Timeout
56733123453	|Success
46733123454	|Pending
....
```python 
Request_Tranfer_Payment("56733123453","SAND BOX","500")
```
