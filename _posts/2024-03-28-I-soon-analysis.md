# I-Soon Internal Documents Leakage Report: Analysis of Wireless Networking Spying Tools and Attacks to Telecommunication Infrastructure

Recently, internal documents and contracts from I-soon, a Chinese company, were leaked and published on GitHub in February, 2024. Subsequently, the leak was confirmed by I-soon's employees.

In brief, I-soon (安洵資訊科技) claims to be a company providing cybersecurity services, including an intelligence platform, security training, and cybersecurity-related solutions. It is headquartered in Shanghai. From the leaked documents, we can glimpse how I-soon provided spying toolkits and conducted hacking activities targeting foreign authorities, including Pakistan, Poland, the UK government. Even some Taiwanese educational institutions are also listed as victims. The customers of I-soon are allegedly the Chinese government.

The spying tools mentioned in the documents support not only Windows and Linux systems but also iOS and Android mobile devices. These spying tools include both software and hardware, such as Windows malware, spyware, and modified WiFi equipment or devices with altered SoCs, enabling the collection of information from confidential files, personal messages, SMS logs, contacts, and even GPS locations.

In this post, we will focus on the technical aspects related to communication networking products and services and share the investigation and hypotheses, including the following sections:

- Unsensible WiFi Hardware
- Wi-Fi Near Field Attack System
- Tele2 users phone calls logs, GPS location exposure
## 1. “無感”(“Unsensible”) WiFi Hardware

![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.001](https://hackmd.io/_uploads/rk3_7BtJA.jpg)

Fig:1

Mentioned in the leaked documentation, I-Soon provides WiFi hardware capable of injecting a backdoor process silently into Android mobile devices that connect to it. The image above illustrates the exploitation scenario. The malicious WiFi hardware is implanted in the WiFi router, and once the router has been brought into the confidential environment, the spying process is started.

The specified features are:

- It can download the backdoor applications with full storage access permission(e.g. MANAGE\_EXTERNAL\_STORAGE, which allows the application to access to most of the shared data), silently to devices connected to the WiFi without displaying any pop-ups.
- After the backdoor is installed, it will upload files, contact, call history and messages from the devices back to the C2 server.
- It can send the packets to C2 servers with 3G/4G network if the ethernet connection is offline.

The document doesn’t explain how the system implements the feature mentioned in Point 1 although it is claimed that MANAGE\_EXTERNAL\_STORAGE permission is risky and needs the user approval after Android 11 in Android Developer Guides.

## 2. Wi-Fi “Near Field Attack System” (“抵近攻擊系統”)

Another product mentioned in the document is a type of spying toolkit networking hardware.

This product consists of a chip implanted either in power banks or plugs, common physical equipment found in offices. The chip comes in two versions: standard and mini. Standard chips are implanted in the power banks and have their own power supply, while mini chips are embedded in the plugs.


![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.002](https://hackmd.io/_uploads/BkeiQBY1R.jpg)


Fig: 2

![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.003](https://hackmd.io/_uploads/SJYo7HF1C.jpg)


Fig: 3

The above images illustrate scenarios in which the standard version and the mini version WiFi near field systems exploit attack victims. Once the fake power bank or plug is placed in the office, it attempts to crack the WiFi network and collect traffic from devices connected to the network. Finally, the collected traffic is sent via 4G network to the remote C2 server.

Here are the features briefly listed:

- The system supports environments using WEP, WPA, WPA2, etc.
- The system can crack the WiFi network when in close physical proximity.
- The system can spoof known SSIDs
- The system can spoof user account hash.
- The system can collect traffic from devices on the same network.
- The system can establish a SOCKS5 proxy tunnel to send packets to the remote server.
- The system can self-destruct upon receiving a remote command when the device is in an insecure situation.

The document doesn't detail how the attack exploits the WiFi system to obtain credential information from victims. The possibility of holding zero-days as useful weapons cannot be denied. Nevertheless, hypotheses can still be drawn from the mentioned behaviors and features.

The easiest way to explain the first behavior is that the chip actively attempts to connect to the WiFi router and brute-force its password and it has been mentioned in the leaked document as one of the methods they compromise the WiFi router. However, considering the system's

capability to support WPA and WPA2 (point 1), cracking the WiFi network, especially in an office scenario with WPA-PSK protection, would be challenging.

In searching for similar vulnerabilities based on the described behaviors, CVE-2023-52160 and CVE-2023-52161 fit the descriptions.

In the case of CVE-2023-52160, it is a `wpa_supplicant` vulnerability when victims lacking proper WiFi connection settings might connect to a network with the same SSID but with the wrong TLS certificate. The attacker tricks the victim into connecting to the clone session, thereby gaining the victim's credentials. The flowchart is shown in the following image shown the exploit process. The attacker’s AP will disguise with the SSID which is known by the victim, and the victims will response their identity(IDs and passwords) to the attacker. In the normal situation, the victims should verify if the TLS certificate is trustworthy or not then the TLS session established at Stage 2, but with the misconfiguration, the victims will connect to the attacker over the clone session with the different certificate toward the attacker. At this point, the attacker is able to gain the credentials from the victims, including their identity and the their traffic.

![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.004](https://hackmd.io/_uploads/SyN27HK1R.jpg)


Fig: 4

The statement regarding CVE-2023-52160 is limited to cases where the victim's device is misconfigured and is near the attacker's access point (AP). However, CVE-2023-52161 affects most Linux-based WiFi routers.

![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.005](https://hackmd.io/_uploads/B1nnXHKJR.jpg)


Fig: 5

With the insecure implementation of the 4-way handshake authentication in Linux-based WiFi routers, as shown in the image above, in the case of `CVE-2023-52161`, the attacker can gain access to the network with zero key by skipping msg2 in the 4-way handshake, which is a message contains `SNounce`(the random number generated by the supplicant) and the `MIC`(Message Integrity Check, a value that shows the integrity of messages) that can be verified by `PTK`(Partial Transition Key) held both in the router and the supplicant, thereby bypassing authentication and accessing the router.

The reason why we hypothesize that they adopted this type of vulnerabilities for implementation in the WiFi near field system is that if the victims have subscribed to **SMS over IP(SMSoIP)** services, then the messages might be sent over Internet Protocol. Moreover, in a scenario where the WiFi AP is supported with a cellular network connection, those devices might have an SMS server (daemon) to handle incoming and outgoing messages, which is also a possible exploitable statement if the WiFi AP has been compromised.

Below is a screenshot from the documents, that purportedly shows how the hardware looks in real life. The first picture resembles a Xiomi powerbank, the second might be a MTK chip.


![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.006](https://hackmd.io/_uploads/SkqT7SFyR.jpg)


Fig: 6

## Tele2 Users Phone Calls Logs, GPS Location Exposure

We also found that there are some \*.csv logs are from victims who are customers of Tele2, a Kazakhstani telecom company, recording the spying activities, leaking the residential information and credentials. These logs show the potential risk when the telecommunication infrastructures being compromised.

![Aspose.Words.2bf3c709-941f-432a-857b-31ae838d886e.007](https://hackmd.io/_uploads/HJL0XHFyC.jpg)


Fig: 7

The first CSV records the phone call durations, along with the **Location Area Code (LAC)** and the **cell ID** that accepted the signals of those phone calls. **Public Land Mobile Network(PLMN)** is separated to many **Location Area(LA)**, and each LA is labeled with **Location Area Identity(LAI)**. LAI contains 3 different codes, and Location Area Code(LAC) is one of it. LAC indicates a specific area with several cells, with the information of LAC and the Cell ID, it is possible to confirm the physical location from which the callers sent out their phone calls. Given that the Country Code of the numbers is +7 (highlighted with the red box) and the IP address of the M**obile Switching Center (MSC_ID)**, we can conclude that these calls originate from Kazakhstan.

Moreover, in the second CSV file, it records the personal identities of the phone numbers, including names, addresses, the place where the victims applied for their Kazakhstan passports, their places of employment, and their dates of birth.

There may be a China-Kazakhstan geopolitical angle involved in the selection of victims. A possible explanation for collecting the credential information and contacts of Kazakh victims relates to protest activities in Xinjiang. Specifically, the information pertains to Uyghur protesters involved in the protests who are now living in Kazakhstan.

### References

We are grateful for the materials and the comprehensive reports from other researchers listed below, which enriched our research:

- [https://github.com/I-S00N/I-S00N ](https://github.com/I-S00N/I-S00N)(repo discarded)
- [https://thehackernews.com/2024/02/new-wi-fi-vulnerabilities-expose.html ](https://thehackernews.com/2024/02/new-wi-fi-vulnerabilities-expose.html)
- [https://www.top10vpn.com/research/wifi-vulnerabilities/ ](https://www.top10vpn.com/research/wifi-vulnerabilities/)
- [https://nvd.nist.gov/vuln/detail/CVE-2023-52160 ](https://nvd.nist.gov/vuln/detail/CVE-2023-52160)
- [https://nvd.nist.gov/vuln/detail/CVE-2023-52161 ](https://nvd.nist.gov/vuln/detail/CVE-2023-52161)
- [https://git.kernel.org/pub/scm/network/wireless/iwd.git/commit/?id=6415420f1c92012f6 4063c131480ffcef58e60ca ](https://git.kernel.org/pub/scm/network/wireless/iwd.git/commit/?id=6415420f1c92012f64063c131480ffcef58e60ca)
- [https://harfanglab.io/en/insidethelab/isoon-leak-analysis/ ](https://harfanglab.io/en/insidethelab/isoon-leak-analysis/)
- [https://www.trendmicro.com/zh_tw/research/24/b/earth-lusca-uses-geopolitical-lure-to- target-taiwan.html ](https://www.trendmicro.com/zh_tw/research/24/b/earth-lusca-uses-geopolitical-lure-to-target-taiwan.html)
- [https://thehackernews.com/2024/02/new-wi-fi-vulnerabilities-expose.html ](https://thehackernews.com/2024/02/new-wi-fi-vulnerabilities-expose.html)