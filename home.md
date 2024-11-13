# <b>MoMo Open API SandBox - Jupyter Note book </b>

## Table of Contents
1. [Initialization](#initialization)
   - [timer-wait-time function](#timer-wait-time-function)
2. [Authorization](#authorization)
   - [Creating API User on the SandBox](#creating-api-user-on-the-sandbox)
   - [Creating the API Key Password For the API User](#creating-the-api-key-password-for-the-api-user)
   - [Generate Access Baerer Token](#generate-access-baerer-token)
   - [Bearer Token expiry validation function](#bearer-token-expiry-validation-function)
3. [Get Paid](#get-paid)
   - [Debit API Function](#debit-api-function)
   - [Debit Status API Function](#debit-status-api-function)
   - [Notification to the Payer after a successful Debit Request](#notification-to-the-payer-after-a-successful-debit-request)
   - [Refund of a successful Debit Partial or full](#refund-of-a-successful-debit-partial-or-full)
   - [Get Status of a Refund Request](#get-status-of-a-refund-request)
   - [Test GetPaid Functions Status](#test-getpaid-functions-status)
4. [Fetch Customer Details KYC](#fetch-customer-details-kyc)
   - [Get Basic Info KYC Function](#get-basic-info-kyc-function)
   - [Get Detailed KYC Function With Consent](#get-detailed-kyc-function-with-consent)
     - [Generate Consent bc-authorize Function](#generate-consent-bc-authorize-function)
     - [Generate Consent Token oauth2 Function](#generate-consent-token-oauth2-function)
     - [Get Detailed KYC with Consent Token oauth2 Function](#get-detailed-kyc-with-consent-token-oauth2-function)
     - [Test detailed KYC Functions](#test-detailed-kyc-functions)
5. [Pay](#pay)
   - [Disburse Transfer Function](#disburse-transfer-function)
   - [Get Status on Disbursements Transfer Function](#get-status-on-disbursements-transfer-function)
   - [Test Pay Functions Disbursement](#test-pay-functions-disbursement)
6. [Distribute](#distribute)
   - [CashIn Deposit Function](#cashin-deposit-function)
   - [CashIn Deposit Status Function](#cashin-deposit-status-function)
   - [CashOut Request To Withdraw Function](#cashout-request-to-withdraw-function)
   - [CashOut Request Status Function](#cashout-request-status-function)
   - [Test CashIn and CashOut Functions](#test-cashin-and-cashout-functions)
. [Invoice](#invoice)
   - [Create Invoice](#create-invoice)
   - [Check Invoice Status](#check-invoice-status)
   - [Cancel Invoice Request](#cancel-invoice-request)
   - [Test Invoice Functions](#test-invoice-functions)


# <b>MoMo Open API SandBox - Jupyter Note book </b>

## <b>Intialization</b>
...
```python
from datetime import datetime
global Api_User 
global Api_Key 
GetPaid_Debit_Request_Ref_ID = "" #UUID String Request Reference 
Pay_Transfer_Request_Ref_ID = "" #UUID String Request Reference
CashIn_Request_Ref_ID = "" #UUID String Request Reference
CashOut_Request_Ref_ID = "" #UUID String Request Reference
Invoice_Request_Ref_ID = "" #UUID String Request Reference
Invoice_Delete_Request_Ref_ID = "" #UUID String Request Reference
Token=""
Token_expiry_time = ""
Token_expired = False
Token_expiry_time = datetime.now()
Environment = "sandbox" #Target Environment  
Collection_Subscription_Primary_Key  = "4c91dae7a6f1474387a23a1f3d448eb7"#Primary Key for Collection Subscription.https://momodeveloper.mtn.com/profile
Disbursement_Subscription_Primary_Key  = "dec90f29f4e14137912bfc3236d51cbe"#Primary Key for Disbursement Subscription.https://momodeveloper.mtn.com/profile
Base_Url = "https://sandbox.momodeveloper.mtn.com" #SandBox Base URL
```

### <b>timer-wait-time funtion</b>
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
# <b>Authorization</b>
## <b>Creating API User on the SandBox</b>
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
```
## <b>Creating the API Key Password For the API User</b>        

```python
#Create_API_User_SandBox()
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
Test the APIUser and Key funtion to create them
Create_API_User_SandBox() #Function create to create API User and store value in Variable {{Api_User}}
Create_API_Key_SandBox() #Function to create API Key(Password) for the Api User Created and stored in variable {{Api_Key}}
```
## <b>Generate Access Baerer Token</b>
<b>Bearer Token is generated useing encoding API User and API Key Base64. Token authenticats most API requests,<br>
NOTE: Each Token expiry is 3600 seconds from time of creation.</b>

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
## <b>Bearer Token expiry validation funtion</b>
<b>it is advisable to verify the token's validity and only generate a new one when the previous token has expired.</b>

```python
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
# <b>Get Paid</b>
1. Debit API Function
2. Debit Status API Function
3. Notification to the Payer after a successful Debit Request
4. Refund of a successful Debit Partial or full
5. Get Status of a Refund Request
6. Test GetPaid Functions Status

## <b>Debit API Function</b>
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
      
      #print("Request_Reference_ID :"+GetPaid_Debit_Request_Ref_ID )
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
## <b>Debit Status API Function</b>

```python
#Check Status 
def Check_Debit_Status(X_Reference_Id_Of_The_Debit_Request):
    import requests as rq
    import json
    import traceback
    print("Waiting for Debit Status")
    countdown(6)
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
## <b>Notification to the Payer after a successful Debit Request.</b>

```python
#Send a Notification After a succesfull Request to Pay
def Send_Notification (X_Reference_Id_Of_The_Debit_Request):
    import requests as rq
    import json
    import traceback
    countdown(6)
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
## <b>Refund of a successful Debit Partial or full</b>
```python
#Send a refund of a debit transaction
def Refund_Debit (X_Reference_Id_Of_The_Debit_Request,Amount):
    import requests as rq
    import json
    import uuid
    import traceback
    global GetPaid_Debit_Refund_Ref_ID
    GetPaid_Debit_Refund_Ref_ID = str(uuid.uuid4())
    Token_Status()
    url = Base_Url+"/disbursement/v1_0/refund"
    headers = {
    "X-Reference-Id": GetPaid_Debit_Refund_Ref_ID,
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token
    }
    body = {    
    "amount": str(Amount),
    "currency": "EUR", #use the currency as EUR in the SandBox
    "externalId": str(uuid.uuid1()), #Used for Reconciliation between application and MoMo platform. 
    "payerMessage": "MoMo Reund API", #Message sent to the Payer
    "payeeNote": "MoMo Refund API", #Message Note to the  Payee
    "referenceIdToRefund": str(X_Reference_Id_Of_The_Debit_Request)
  }
    
    try:
        resp = rq.request("post", url, json=body, headers=headers)
        if(str(resp.status_code) == "202"):
           print("Refund request Id "+str(X_Reference_Id_Of_The_Debit_Request)+" Amount "+str(Amount)+" "+ "Response Code "+str(resp.status_code))
        elif (str(resp.status_code) == "404"):
           print("Check The Base_URL ")
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)

```
## <b>Get Status of a Refund Request</b>
....
```python
#####
#Check Refund Status
def Check_Refund_Status(X_Reference_Id_Of_The_Refund_Request):
    import requests as rq
    import json
    import traceback
    Token_Status()
    global Refund_Status
    print("Waiting for Refund Status")
    countdown(6)
    url = Base_Url+"/disbursement/v1_0/refund/"+X_Reference_Id_Of_The_Refund_Request
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:        
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        Status_Json_DD = str(Status_Json).replace('\'', '"')
        Refund_Status = Status_Json.get('status')
        print(str(Status_Json))
        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
    
#Check_Status("")
```
### <b>Test GetPaid Functions Status</b>
<b>Note: Responses with payer_Id_msisdn as the MSISDN </b>
<b>| payer_id-Msisdn | Response status </b>
| --- | --- | 
46733123450 |Failed 
46733123451 |Rejected 
46733123452 |Timeout 
56733123453 |Success 
46733123454 |Pending

....
```python
#####
#Test the Debit, Notification and the Refund
Request_Debit_Payment("56733123452","50000")
status = Check_Debit_Status(GetPaid_Debit_Request_Ref_ID)
if (Debit_Status == "SUCCESSFUL"):
    
    Send_Notification(GetPaid_Debit_Request_Ref_ID)
    Refund_Debit(GetPaid_Debit_Request_Ref_ID,"200")
    
    Check_Refund_Status(GetPaid_Debit_Refund_Ref_ID)

else:
    print(Debit_Status+""+" Notification No Sent")
```
....


# <b>Fetch Customer Details KYC</b>
1. <b>Get Basic Info KYC Function</b>
2. <b>Get Detailed KYC Function With Consent</b>
3. <b>Generate Consent bc-authorize Function</b>
4. <b>Generate Consent Token oauth2 Function</b>
5. <b>Status Token oauth2 Function</b>
6. <b>Get Detailed KYC with Consent Token oauth2 Function</b>
7. <b>Test detailed KYC Functions</b>
8. <b>Get Basic Info KYC Function</b>

....
## <b>Get Basic Info KYC Function</b>
```python
#####
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
```
....
```python
#####
#Check_BasicInfo_KYC("56733123453","BOX SAND")
```
## <b>Get Detailed KYC Function With Consent</b>
### <b>Generate Consent bc-authorize Function</b>

....
```python
#####
#Send Consent to customer for Approval, generating unique Consent ID as auth_req_id
def bc_authorize (Customer_Number_):
    import requests as rq
    import json
    import traceback
    global Consent_ID, Customer_Number
    Customer_Number = str(Customer_Number_)  
    Token_Status()
    url = Base_Url+"/disbursement/v1_0/bc-authorize"
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Disbursement_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    'Content-Type': 'application/x-www-form-urlencoded'
    }
    body = 'scope=all_info&login_hint=ID:'+Customer_Number+'/MSISDN&access_type=offline' 
    #scope will differ based on the information you want to access.   
    try:
        resp = rq.request("post", url, data=body, headers=headers)
        resp_json = resp.json()
        Consent_ID =resp_json.get("auth_req_id")
        #print(Consent_ID )
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
#bc_authorize ("56733123453")
```
### <b>Generate Consent Token oauth2 Function</b>

....
```python
#####
#Function to generate Oauth Token and set Token expiry. 
def Get_Oauth_Token(consent_auth_req_id):# function to return token (renews token if expired)
    import requests as rq
    import traceback
    import json
    from datetime import datetime, timedelta
    global Token_Oauth
    global Disbursement_Subscription_Primary_Key
    global Token_Oauth_expiry_time
    Token_Oauth_expiry_time = datetime.now()
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
Get_Oauth_Token(Consent_ID)
New Token Generated Expiring at :2024-11-10 16:53:57.056723
Status Token oauth2 Function
#Function to Validate  Status of Token
#If the Token is Expired a new one will be generated. 
def Token_Oauth_Status():
    #from datetime import datetime, timedelta
    #Token_Oauth_expiry_time = datetime.now()
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
### <b>Get Detailed KYC with Consent Token oauth2 Function</b>
```python
#Get Detailed Info KYC (FirstName, LastName, gender, DoB, ID_Number)
def Get_DetailedInfo_KYC():
    import requests as rq
    import json
    import traceback
    #bc_authorize ("56733123453")
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
#Get_DetailedInfo_KYC()
```
....

#### <b>Test detailed KYC Functions</b>
<b>Note: Response is the same with any provided Customer_msisdn as the MSISDN in sand Box</b>

....
```python
#####
bc_authorize ("6575688")
Get_DetailedInfo_KYC()
```
<b>Possible configurable  KYC </b>
{"sub":"0","name":"Sand Box","given_name":"Sand","family_name":"Box","birthdate":"1976-08-13","locale":"sv_SE","status":"ACTIVE","gender":"MALE","email":"email@domain.test","email_verified":true,"phone_number":"46123456789","phone_number_verified":true,"address":{"formatted":"Street 17\n123 45 Karlskrona\nBlekinge\nSweden","street_address":"Street 17","postal_code":"123 45","locality":"Karlskrona","region":"Blekinge","country":"Sweden"},"updated_at":1731242938,"credit_score":123,"active":true,"country_of_birth":"Sweden","region_of_birth":"Blekinge","city_of_birth":"Karlskrona","occupation":"Manager","employer_name":"Ericsson","identification_type":"PASS","identification_value":"S1234567"}

....

#####
# <b>Pay</b>
1. <b>Disburse Transfer Function</b>
2. <b>Get Status on Disbursements Transfer Function</b>
3. <b>Test Pay Functions Disbursement</b>
4. <b>Disburse Transfer Function</b>

## <b>Disburse Transfer Function</b>

```python
#####
#Function that initiates a Transfer from  Business  Wallet to a customer's  
def Request_Tranfer_Payment(MSISDN,MSISDN_NAME,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global Pay_Transfer_Request_Ref_ID 
  Token_Status()
  Check_BasicInfo_KYC(MSISDN,MSISDN_NAME)# Function that Validates the Names of the Customer(Receiving Party)
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
      "payerMessage": "MoMo Transfer API", #Message sent to the Payer
      "payeeNote": "MoMo Transfer API" #Message Note to the  Payee
    }
    try:
      resp = rq.request("post", url, json=body, headers=headers)
      if(str(resp.status_code) == "202"):
        print("Transfer request "+Pay_Transfer_Request_Ref_ID+" to MSISDN "+MSISDN+" Amount "+Amount+" "+ "Response Code "+str(resp.status_code))
        #print("Completed")        
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
## <b>Get Status on Disbursements Transfer Function</b>
....
```python
#####
#Check Status Disbursements Transfer
def Check_Status_Transfer(X_Reference_Id_Of_The_Transfer_Request):
    import requests as rq
    import json
    import traceback
    print("Waiting for Disbursements Transfer Status")
    countdown(6)
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
### <b>Test Pay Functions Disbursement</b>
....
<b>Note: Responses with payee_Id_msisdn as the MSISDN </b>
| <b>payee_id-Msisdn | Response status </b>
| --- | --- | 
46733123450 |Failed 
46733123451 |Rejected 
46733123452 |Timeout 
56733123453 |Success 
46733123454 |Pending

```python
#Function to initiate a Transfer from Business Wallet to Customer Wallet
#Function validates Customer Names before initiating the Transfer(SandBox names are reversed)
Request_Tranfer_Payment("56733123453","SAND BOX","500") 
#Function to check Transfer Status after 6 seconds
Check_Status_Transfer(Pay_Transfer_Request_Ref_ID)
```

# <b>Distribute</b>
1. CashIn Deposit Function
2. CashIn Deposit Status Function
3. CashOut Request To Withdraw Function
4. CashOut Request Status Function
5. Test CashIn and CashOut Functions
6. Test CashIn Functions
7. Test CashOut Functions
8. CashIn Deposit Function


## <b>CashIn Deposit Function</b>
<b>With Customer Names Validations</b>
....
```python
#CashIn Deposit 
#####
#Function that initiates a CashIn  from  Business  Wallet to a customer's  
def CashIn_Tranfer_Deposit(MSISDN,MSISDN_NAME,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global CashIn_Request_Ref_ID 
  Token_Status()
  Check_BasicInfo_KYC(MSISDN,MSISDN_NAME)# Function that Validates the Names of the Customer(Receiving Party)
  if(Proceed==True):
    CashIn_Request_Ref_ID   = str(uuid.uuid4())  
    url = Base_Url+"/disbursement/v1_0/deposit"
    headers = {
      "X-Reference-Id": CashIn_Request_Ref_ID , #Unique for every request, used to validate status of the request. 
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
      "payerMessage": "MoMo CashIN API", #Message sent to the Payer
      "payeeNote": "MoMo CashIN API" #Message Note to the  Payee
    }
    try:
      resp = rq.request("post", url, json=body, headers=headers)
      if(str(resp.status_code) == "202"):
        print("CashIn "+CashIn_Request_Ref_ID+" to MSISDN "+MSISDN+" Amount "+Amount+" "+ "Response Code "+str(resp.status_code))
        #print("Completed")        
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
## <b>CashIn Deposit Status Function</b>
....
```python
#####
#Check Status CashIn Status
def Check_Status_CashIn(X_Reference_Id_Of_The_CashIn_Request):
    import requests as rq
    import json
    import traceback
    print("Waiting for CashIn  Status")
    countdown(6)
    Token_Status()
    url = Base_Url+"/disbursement/v1_0/deposit/"+X_Reference_Id_Of_The_CashIn_Request
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
    
#Check_Status_CashIn("Check_Status_Transfer")
```
## <b>CashOut Request To Withdraw Function</b>
....
```python
#####
#Function that initiates a CashOut Withdraw from Customer Wallet to a Bussiness
#In return the Business will send the Cash to the Customer
def Request_CashOut_Withdraw(MSISDN,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global CashOut_Request_Ref_ID
  Token_Status()
  CashOut_Request_Ref_ID = str(uuid.uuid4())
  url = Base_Url+"/collection/v1_0/requesttowithdraw"
  headers = {
    "X-Reference-Id": CashOut_Request_Ref_ID, #Unique for every request, used to validate status of the request. 
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
    "payerMessage": "MoMo CashOut API", #Message sent to the Payer
    "payeeNote": "MoMo CashOut API" #Message Note to the  Payee
  }
  try:
    resp = rq.request("post", url, json=body, headers=headers)
    if(str(resp.status_code) == "202"):
      print("CashOut Withdraw request from MSISDN "+MSISDN+" Amount "+Amount+" "+ "Response Code "+str(resp.status_code))

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

## <b>CashOut Request Status Function</b>
....
```python
#Check CashOut Status 
def Check_CashOut_Status(X_Reference_Id_Of_The_CashOut_Request):
    import requests as rq
    import json
    import traceback
    print("Waiting for CashOut Status")
    countdown(6)
    Token_Status()
    global CashOut_Status
    url = Base_Url+"/collection/v1_0/requesttowithdraw/"+X_Reference_Id_Of_The_CashOut_Request
    headers = {
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        #Status_Json_DD = str(Status_Json).replace('\'', '"')
        #CashOut_Status = Status_Json.get('status')
        print(str(Status_Json))
        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)
    
#Check_Status("")
```
### <b>Test CashIn and CashOut Functions</b>
<b>Note: Responses with partyId as the MSISDN </b>
| partyId | Response status 
| --- | --- | 
46733123450 |Failed 
46733123451 |Rejected 
46733123452 |Timeout 
56733123453 |Success 
46733123454 |Pending

....
```python
Test CashIn Functions
#Function to initiate a CashIn from Business Wallet to Customer Wallet
#Function validates Customer Names before initiating the CashIn(SandBox names are reversed)
CashIn_Tranfer_Deposit("56733123453","SAND BOX","50089") 
#Function to check CashIn Status after 6 seconds
Check_Status_CashIn(CashIn_Request_Ref_ID)

```
# <b>Invoice</b>
1. <b>Create Invoice</b>
2. <b>Check Invoice Status</b>
3. <b>Cancel Invoice Request</b>
4. <b>Test Invoice Functions</b>

## <b>Create Invoice</b>
....
```python
#Function that initiates a CashOut Withdraw from Customer Wallet to a Bussiness
#In return the Business will send the Cash to the Customer
def Request_Create_Invoice(intended_MSISDN,MSISDN_Payee,Amount):
  import requests as rq
  from datetime import datetime, timedelta
  import traceback
  import uuid
  global Invoice_Request_Ref_ID
  Token_Status()
  Invoice_Request_Ref_ID = str(uuid.uuid4())
  url = Base_Url+"/collection/v2_0/invoice"
  headers = {
    "X-Reference-Id": Invoice_Request_Ref_ID, #Unique for every request, used to validate status of the request. 
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token, #Avoid creating new tokens for every request,  track the Expiry 
    "Content-Type": "application/json",
    "X-Callback-Url":"https://webhook.site/mycallback/site"# You can add X-Callback-Url to receive the callback  ("X-Callback-Url":"https://webhook.com/mysite/status")
  }
  body = {    
    "amount": Amount,
    "currency": "EUR", #use the currency as EUR in the SandBox
    "externalId": str(uuid.uuid1()), #Used for Reconciliation between application and MoMo platform. 
    "validityDuration": "360", #Duration in seconds before the Invoice expires in Seconds
    "intendedPayer": { #Customer who will pay the Invoice thou not not medentory 
      "partyIdType": "MSISDN",#EMAIL and ALIAS apply as well 
      "partyId": intended_MSISDN
  },
  "payee": {
        "partyIdType": "MSISDN",
        "partyId": MSISDN_Payee
    },
    "description": "MoMo Invoice API", #Message sent to the Payer   
  }
  try:
    resp = rq.request("post", url, json=body, headers=headers)
    if(str(resp.status_code) == "202"):
      print("Invoice No "+Invoice_Request_Ref_ID+" Amount "+Amount+" "+ "Created  Code "+str(resp.status_code))

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
#Request_Create_Invoice("5677876667","5678399378","700000")
```

## <b>Check Invoice Status</b>
....
```python
#Check Invoice Status 
#Function not only checks the status of the invoice and fetch details like invoice number
def Check_Invoice_Status(X_Reference_Id_Of_The_Invoice_Request):
    import requests as rq
    import json
    import traceback
    print("Waiting for Invoice Status")
    countdown(6)
    Token_Status()    
    url = Base_Url+"/collection/v2_0/invoice/"+X_Reference_Id_Of_The_Invoice_Request
    headers = {    
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token,
    }
    try:
        resp = rq.request("get", url,headers=headers)
        Status_Json = resp.json()
        #Status_Json_DD = str(Status_Json).replace('\'', '"')
        
        print(str(Status_Json))        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)    
#Check_Invoice_Status(Invoice_Request_Ref_ID)
```
## <b>Cancel Invoice Request</b>
....
```python
#Delete Invoice Status 
#Function Delete Invoice
def Delete_Invoice_request(X_Reference_Id_Of_The_Invoice_Request):
    import requests as rq
    import uuid
    import traceback
    global Invoice_Delete_Request_Ref_ID, Invoice_Request_Ref_ID
    Token_Status()
    Invoice_Delete_Request_Ref_ID = str(uuid.uuid4())
    url = Base_Url+"/collection/v2_0/invoice/"+X_Reference_Id_Of_The_Invoice_Request
    headers = {
    "X-Reference-Id": Invoice_Delete_Request_Ref_ID,
    "X-Target-Environment": Environment,
    "Ocp-Apim-Subscription-Key": Collection_Subscription_Primary_Key,
    "Authorization":"Bearer "+Token
    }
    body = {    
    "externalId": str(uuid.uuid1())}
    try:
        resp = rq.request("delete", url,json=body,headers=headers)
        #Status_Json = resp.json()
        #Status_Json_DD = str(Status_Json).replace('\'', '"')        
        print(resp.status_code)        
    except:
        print("Something Is Wrong ")
        traceback.print_exc()       
    #print(Status_Json)    
#Delete_Invoice_request(Invoice_Request_Ref_ID)
```
### <b>Test Invoice Functions</b>
Note: Responses with partyId as the MSISDN 
| <b>partyId | Response status </b>
| --- | --- |
46733123450 |Failed 
46733123451 |Rejected 
46733123452 |Timeout 
56733123453 |Success 
46733123454 |Pending

```python
Request_Create_Invoice("5677876667","5678399378","700000")
Check_Invoice_Status(Invoice_Request_Ref_ID)
Delete_Invoice_request(Invoice_Request_Ref_ID)
```
