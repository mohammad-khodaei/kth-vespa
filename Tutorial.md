# Introduction #

The interfaces to connect to VeSPA are elaborated below. You can download the entire interface file from <a href='http://code.google.com/p/kth-vespa/source/browse/trunk/interfaces/interfaces.proto'>here</a>.



&lt;P&gt;


Further details on the main interfaces are discussed here.


&lt;/P&gt;




## Servers Configuration ##


  * LTCA
    * 172.31.212.101
    * LTCA\_SERVER\_URL  = https://172.31.212.101/cgi-bin/ltca
    * LTCA\_METHOD\_NAME = ltca.operate


  * PCA
    * 172.31.212.102
    * PCA\_SERVER\_URL  = https://172.31.212.102/cgi-bin/pca
    * PCA\_METHOD\_NAME = pca.operate

  * RA
    * Private: 192.168.100.4
    * RA\_SERVER\_URL  = https://192.168.100.4/cgi-bin/ra
    * RA\_METHOD\_NAME = ra.operate

<h1><a></a>Request & Response Type Format Definitions<a href='#ReqResDef'></a></h1>


## LTCA ##
```
REQ_X509_CERT_REQ_VEHICLE_TO_LTCA_USING_PROTO_BUFF                  120
RES_ISSUE_X509_CERT_LTCA_TO_VEHICLE_USING_PROTO_BUFF                121
REQ_X509_CERT_VALIDATION_VEHICLE_TO_LTCA_USING_PROTO_BUFF           122
RES_X509_CERT_VALIDATION_LTCA_TO_VEHICLE_USING_PROTO_BUFF           123
REQ_NATIVE_TICKET_VEHICLE_TO_LTCA_USING_PROTO_BUFF                  124
RES_NATIVE_TICKET_LTCA_TO_VEHICLE_USING_PROTO_BUFF                  125
REQ_FOREIGN_TICKET_VEHICLE_TO_LTCA_USING_PROTO_BUFF                 126
RES_FOREIGN_TICKET_LTCA_TO_VEHICLE_USING_PROTO_BUFF                 127
REQ_VOUCHER_VEHICLE_TO_LTCA_USING_PROTO_BUFF                        128
RES_VOUCHER_LTCA_TO_VEHICLE_USING_PROTO_BUFF                        129

```


## PCA ##
```
REQ_NATIVE_PSNYM_CERT_VEHICLE_TO_PCA_USING_PROTO_BUFF               220
RES_NATIVE_PSNYM_CERT_PCA_TO_VEHICLE_USING_PROTO_BUFF               221
NATIVE_PSNYM_CERT_ACK_OK_VEHICLE_TO_PCA_USING_PROTO_BUFF            222
REQ_PSNYM_CRL_VEHICLE_TO_PCA_USING_PROTO_BUFF                       223
RES_PSNYM_CRL_PCA_TO_VEHICLE_USING_PROTO_BUFF                       224
REQ_UPGRADING_SYS_CONFIG_CLIENT_TO_PCA_USING_PROTO_BUFF             225
RES_UPGRADING_SYS_CONFIG_PCA_TO_CLIENT_USING_PROTO_BUFF             226
REQ_PSNYM_OCSP_VEHICLE_TO_PCA_USING_PROTO_BUFF                      227
RES_PSNYM_OCSP_PCA_TO_VEHICLE_USING_PROTO_BUFF                      228
```



&lt;BR&gt;


## Interfaces ##
Google Protocol Buffer is used to connect to the servers. In the following, you can find the interfaces to the servers for different services. **Take into consideration that the "required fields" are required to be fulfilled. Otherwise, the serialization will be failed on the server.**

### General Info for the Interfaces ###
Here, you can find information regarding the fields:


&lt;BR&gt;



| **Fields** | **Values** |
|:-----------|:-----------|
| PCA\_ID | 1001 |
| LTCA\_ID | 1002 |
| RA\_ID | 1003 |
| iNonce | 0-65535 |
| tTimeStamp | long int |



&lt;BR&gt;


Nonce is an Integer random value (0-65535). Time-stamp identifies also the current time-stamp when the request is sent.



&lt;BR&gt;


### Voucher ###
Obtaining an X.509 certificate requires getting a voucher from LTCA. Having sent the voucher request to the server, a valid voucher is sent to your kth email, be used in obtaining X.509 certificate.

**NOTE: Each email address can be used to obtain a voucher. Store the generated private key when the request is sent. If you lost your certificate, you can contact the admins of the Wiki to receive your certificate. Later on, you can also fetch your certificate from LDAP directory. (LDAP is not supported yet.)**

The voucher request format is defined as below:
```
message msgVoucherReq_V2LTCA {
	required int32 iReqType = 1 [default = -1];
        required string strUserName = 2 [default = ""];
        required string strPwd = 3 [default = ""];
	required string strEmailAddress = 4 [default = ""];
	required string strCaptcha = 5 [default = ""]; 
	required int32 iNonce = 6 [default = -1];
	required int64 tTimeStamp = 7 [default = -1];
};
```

Leave the username and password field empty. Moreover, set 'strCaptcha' to "captcha".

The voucher response format is as below:
```
message msgVoucherRes_LTCA2V {
	required int32 iReqType = 1 [default = -1];
        optional msgSignerInfo signer = 2;
	required string strVoucher = 3 [default = ""];
	required int32 iNonce = 4 [default = -1];
	required int64 tTimeStamp = 5 [default = -1];
	required msgError stErrInfo = 6;
	optional msgSignature stSign = 7;
};
```




&lt;BR&gt;


### X509 Certificate ###
Having received a voucher, one can receive an X.509 certificate. Creating the certificate signing request requires to add the same email address inside.

The X.509 certificate request format is as below:
```
message msgX509CertReq_V2LTCA {
	required int32 iReqType = 1 [default = -1];
	required int32 iLTCAIdRange = 2 [default = -1];
	required string strProofOfPossessionVoucher = 3 [default = ""];
	required string strX509CertReq = 4 [default = ""];
	required int32 iNonce = 5 [default = -1];
	required int64 tTimeStamp = 6 [default = -1];
};
```

Fulfill "strProofOfPossessionVoucher" with the Voucher, sent to your email. iLTCAIdRange is 1002. strX509CertReq is the X.509 certificate signing request.


&lt;BR&gt;



The response format to receive X.509 certificate is defined as:
```
message msgX509CertRes_LTCA2V {
	required int32 iReqType = 1 [default = -1];
        optional msgSignerInfo signer = 2;
	required string strReqIdentification = 3 [default = ""];
	required string strX509Cert = 4 [default = ""];
	required int32 iNonce = 5 [default = -1];
	required int64 tTimeStamp = 6 [default = -1];
	required msgError stErrInfo = 7;
	optional msgSignature stSign = 8;
};
```

**NOTE: For the time being, LTCA supports issuing X.509 certificates with Elliptic Curve Digital Signature Algorithm (ECDSA). Take into consideration that X.509 certificates with RSA keys are not supported by LTCA. For further understanding on differences between ECDSA and RSA, you can read**<a href='http://www.ijcaonline.org/volume2/number2/pxc387876.pdf'>"Implementation of Elliptic Curve Digital Signature Algorithm"</a> or <a href='http://palms.ee.princeton.edu/PALMSopen/potlapally03analyzing.pdf'>"Analyzing the Energy Consumption of Security Protocols"</a>. Openssl and Bouncy Castle support ECDSA with appropriate curve. You can use NID\_X9\_62\_prime256v1 for the curve, if you use openssl.



&lt;BR&gt;


### Ticket ###
VeSPA is a ticket-based system, equipping users to obtain their desired services using a valid ticket. Moreover, it provides Single-Sign-On (SSO) scheme so that users can get authenticated once and use the rest of the services, accordingly.

The format of a ticket is defined as below:

```
message msgTicketReq {
	required int32 iReqType = 1 [default = -1];
	optional msgSignerInfo signer = 2;
	required uint32 uiServices = 3 [default = 0];
	required uint32 uiPsnymCertNoRequest = 4 [default = 0];
	required int32 iLTCAIdRange = 5 [default = -1];
	required int32 iPCAIdRange = 6 [default = -1];
	required int32 iNonce = 7 [default = -1];
	required int64 tTimeStamp = 8 [default = -1];
        required string strX509Cert = 9[default = ""];
	optional msgSignature stSign = 10;
};
```

The response of a ticket is as follows:

```
message msgTicketRes {
	required int32 iReqType = 1 [default = -1];
	optional msgSignerInfo signer = 2;
	required int32 iTicketSize = 3 [default = -1];
	required string strTicket = 4 [default = ""]; 
	required uint32 uiMaxNoPsnymCert = 5 [default = 0];
	required int32 iLTCAIdRange = 6 [default = -1];
	required int32 iPCAIdRange = 7 [default = -1];
	required int32 iNonce = 8 [default = -1];
	required int64 tTimeStamp = 9 [default = -1];
	required msgError stErrInfo = 10;
	optional msgSignature stSign = 11;
};
```


Tickets are processed by the appropriate service providers while users are not involved in the process. Each ticket has the following format:

```
message msgTicketFormat {
	required int32 iTicketType = 1 [default = -1];
	optional msgSignerInfo signer = 2;
	required string strTicketSerialNo = 3 [default = ""];
	required string strTicketIdentifiableKey = 4 [default = ""];
	required int32 iLTCAIdRange = 5 [default = -1];
	required int32 iPCAIdRange = 6 [default = -1];
	required uint32 uiMaxNoPsnymCert = 7 [default = 0];
	required msgVehicleAttributes stVehicleRole = 8;
	required bool bIsForeignTicket = 9 [default = false];
	required int64 tTicketStartTime = 10 [default = -1];
	required int64 tTicketLifeTime = 11 [default = -1];
	required int64 tPsnymStartTime = 12 [default = -1];
	required int64 tPsnymExpiryTime = 13 [default = -1];
	optional msgSignature stSign = 14;
};
```





&lt;BR&gt;


### Pseudonym ###
Pseudonymous certificate is considered a conditional anonymous certificate, used to protect legitimate users' privacy. Users obtain the services, as X.509 certificate, without revealing their real identity. In order to obtain a pseudonymous certificate, the request should be sent to the PCA as the following format:

```
message msgPsnymCertReq_V2PCA {
	required int32 iReqType = 1 [default = -1];
	required int32 iTicketSize = 2 [default = -1];
	required string strTicket = 3 [default = ""];
	required int32 iLTCAIdRange = 4 [default = -1];
	required int32 iPCAIdRange = 5 [default = -1];
	required int32 iLocation = 6 [default = -1];
	required uint32 uiPsnymCertNo = 7 [default = 0];
	repeated msgWAVECertificateRequest pstPsnymCertReq = 8;
	required int32 iNonce = 9 [default = -1];
	required int64 tTimeStamp = 10 [default = -1];
};
```

The response from PCA to the users is as follows:

```
message msgPsnymCertRes_PCA2V {
	required int32 iReqType = 1 [default = -1];
        optional msgSignerInfo signer = 2;
	required string strReqIdentification = 3 [default = ""];
	required int32 iLTCAIdRange = 4 [default = -1];
	required int32 iPCAIdRange = 5 [default = -1];
	required uint32 uiPsnymCertNo = 6 [default = 0];
	repeated msgPsnymCertFormat stPsnymCert = 7;
	required int32 iNonce = 8 [default = -1];
	required int64 tTimeStamp = 9 [default = -1];
	required msgError stErrInfo = 10;
	optional msgSignature stSign = 11;
};
```


You can download the entire interface file from <a href='http://code.google.com/p/kth-vespa/source/browse/trunk/interfaces/interfaces.proto'>here</a>.