
# Website certificate by Zabbix agent 2 active

## Overview

The template to monitor TLS/SSL certificate on the website by Zabbix agent 2 that works without any external scripts.
Zabbix agent 2 with the WebCertificate plugin requests certificate using the web.certificate.get key and returns
JSON with certificate attributes.

## Requirements

Zabbix version: 7.4 and higher.

## Tested versions

This template has been tested on:
- Website TLS/SSL certificate

## Configuration

> Zabbix should be configured according to the instructions in the [Templates out of the box](https://www.zabbix.com/documentation/7.4/manual/config/templates_out_of_the_box) section.

## Setup

1\. Setup and configure zabbix-agent2 with the WebCertificate plugin.

2\. Test availability: `zabbix_get -s <zabbix_agent_addr> -k web.certificate.get[<website_DNS_name>]`

3\. Create a host for the TLS/SSL certificate with Zabbix agent interface.

4\. Link the template to the host.

5\. Customize the value of {$CERT.WEBSITE.HOSTNAME} macro.

### Macros used

|Name|Description|Default|
|----|-----------|-------|
|{$CERT.EXPIRY.WARN}|<p>Number of days until the certificate expires.</p>|`7`|
|{$CERT.WEBSITE.HOSTNAME}|<p>The website DNS name for the connection.</p>|`<Put DNS name>`|
|{$CERT.WEBSITE.PORT}|<p>The TLS/SSL port number of the website.</p>|`443`|
|{$CERT.WEBSITE.IP}|<p>The website IP address for the connection.</p>||

### Items

|Name|Description|Type|Key and additional info|
|----|-----------|----|-----------------------|
|Get|<p>Returns the JSON with attributes of a certificate of the requested site.</p>|Zabbix agent (active)|web.certificate.get[{$CERT.WEBSITE.HOSTNAME},{$CERT.WEBSITE.PORT},{$CERT.WEBSITE.IP}]<p>**Preprocessing**</p><ul><li><p>Discard unchanged with heartbeat: `6h`</p></li></ul>|
|Validation result|<p>The certificate validation result. Possible values: valid/invalid/valid-but-self-signed</p>|Dependent item|cert.validation<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.result.value`</p></li></ul>|
|Last validation status|<p>Last check result message.</p>|Dependent item|cert.message<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.result.message`</p></li></ul>|
|Version|<p>The version of the encoded certificate.</p>|Dependent item|cert.version<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.version`</p></li></ul>|
|Serial number|<p>The serial number is a positive integer assigned by the CA to each certificate. It is unique for each certificate issued by a given CA. Non-conforming CAs may issue certificates with serial numbers that are negative or zero.</p>|Dependent item|cert.serial_number<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.serial_number`</p></li></ul>|
|Signature algorithm|<p>The algorithm identifier for the algorithm used by the CA to sign the certificate.</p>|Dependent item|cert.signature_algorithm<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.signature_algorithm`</p></li></ul>|
|Issuer|<p>The field identifies the entity that has signed and issued the certificate.</p>|Dependent item|cert.issuer<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.issuer`</p></li></ul>|
|Valid from|<p>The date on which the certificate validity period begins.</p>|Dependent item|cert.not_before<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.not_before.timestamp`</p></li></ul>|
|Expires on|<p>The date on which the certificate validity period ends.</p>|Dependent item|cert.not_after<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.not_after.timestamp`</p></li></ul>|
|Subject|<p>The field identifies the entity associated with the public key stored in the subject public key field.</p>|Dependent item|cert.subject<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.subject`</p></li></ul>|
|Subject alternative name|<p>The subject alternative name extension allows identities to be bound to the subject of the certificate.  These identities may be included in addition to or in place of the identity in the subject field of the certificate.  Defined options include an Internet electronic mail address, a DNS name, an IP address, and a Uniform Resource Identifier (URI).</p>|Dependent item|cert.alternative_names<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.alternative_names`</p></li></ul>|
|Public key algorithm|<p>The digital signature algorithm is used to verify the signature of a certificate.</p>|Dependent item|cert.public_key_algorithm<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.x509.public_key_algorithm`</p></li></ul>|
|Fingerprint|<p>The Certificate Signature (SHA1 Fingerprint or Thumbprint) is the hash of the entire certificate in DER form.</p>|Dependent item|cert.sha1_fingerprint<p>**Preprocessing**</p><ul><li><p>JSON Path: `$.sha1_fingerprint`</p></li></ul>|

### Triggers

|Name|Description|Expression|Severity|Dependencies and additional info|
|----|-----------|----------|--------|--------------------------------|
|Certificate: SSL certificate is invalid|<p>SSL certificate has expired or it is issued for another domain.</p>|`find(/Website certificate by Zabbix agent 2 active/cert.validation,,"like","invalid")=1`|High||
|Certificate: SSL certificate expires soon|<p>The SSL certificate should be updated or it will become untrusted.</p>|`(last(/Website certificate by Zabbix agent 2 active/cert.not_after) - now()) / 86400 < {$CERT.EXPIRY.WARN}`|Warning|**Depends on**:<br><ul><li>Certificate: SSL certificate is invalid</li></ul>|
|Certificate: Fingerprint has changed|<p>The SSL certificate fingerprint has changed. If you did not update the certificate, it may mean your certificate has been hacked. Acknowledge to close the problem manually.<br>There could be multiple valid certificates on some installations. In this case, the trigger will have a false positive. You can ignore it or disable the trigger.</p>|`last(/Website certificate by Zabbix agent 2 active/cert.sha1_fingerprint) <> last(/Website certificate by Zabbix agent 2 active/cert.sha1_fingerprint,#2)`|Info|**Manual close**: Yes|

## Feedback

Please report any issues with the template at [`https://support.zabbix.com`](https://support.zabbix.com)

You can also provide feedback, discuss the template, or ask for help at [`ZABBIX forums`](https://www.zabbix.com/forum/zabbix-suggestions-and-feedback)

