---
layout: single
title:  "Implementing NIST CSF 2.0 in a homelab environment"
author_profile: true
toc: true
related: true
classes: wide
mermaid: true
tags: 
    - grc
header:
  teaser: /assets/images/CSF_Six_Functions.jpg 
---

# Implementing NIST CSF 2.0 in a homelab environment

In the over 6 years I've worked in the field of cybersecurity, I am still surprised by the many threats and vulnerabilities that are out there. As a bug bounty hunter working from the offensive side, I've found insecure APIs in banks that leaked account details and social security numbers, exposed cloud storage buckets leaking source code, and many other high impact vulnerabilities in web applications. But now, in my new role as an analyst and threat hunter in a SOC, I've come to appreciate another battleground that hits closer to home than my targets of the past: end user devices.

On one day I may be alerting a client about a user that fell for a fake captcha and installed an infostealer. The next day it may be a device that is suddenly connecting to a threat actor's command & control domain or is abusing a LOLbin (Living Off the Land Binary) like wscript.exe to download and execute malicious code. The worst of the incidents I handle involve ransomware attacks that sometimes leave companies paralyzed, especially if their security policies were weak or non-existent.

All of this experience has gotten me thinking about my own personal security posture. Like most people in the 21st century, I use computers for both business and pleasure. But I also maintain a homelab that I use for self-hosted services, malware analysis, penetration testing and other activities. I have a desktop and laptops that I use for work, shopping, banking, and occasional gaming. How do I even begin to try and secure all of this? Well the answer that I have found involves security frameworks which are collections of policies, practices, and procedures designed to establish a strong cybersecurity posture and reduce risk. Specifically, I've chosen to be guided by and to implement the **NIST Cybersecurity Framework 2.0**. As we will see, I'll be taking off my technical hat and entering the world of governance, risk, and compliance (GRC).

## Why cybersecurity professionals should implement NIST CSF 2.0

![NIST CSF 2.0 Core Functions](/assets/images/CSF_Six_Functions.jpg)

*NIST CSF 2.0 Core Functions*

Security frameworks like ISO-27001, NIST CSF 2.0, PCI DSS, and others are typically used by organizations to help them manage risk, protect data, and maintain compliance. However, even a freelancer or small business owner can benefit from them. Particularly helpful with NIST CSF 2.0 is the fact that it focuses on outcomes rather than strict controls like other frameworks. This makes it flexible and a good springboard for me into GRC. NIST CSF 2.0 components include the **Core**, **Organizational Profiles**, and **Tiers**. The Core is organized into six functions which will serve as the foundations of my personal cybersecurity program. New to CSF 2.0 is the **Govern (GV)** function, which is going to help me clarify what outcomes I'm hoping to achieve with the other five functions and how to prioritize them. 

### Govern : Defining the 'Why?'

Now in a corporate environment a business mission statement would typically be used to help guide this part of the process. But what about in my case as a security researcher? Do I have a specific mission? Without any goals defined it is impossible to define the risks. After some thought I came up with the following mission statement:

*"**To help secure cyberinfrastructure by identifying and reporting vulnerabilities in systems through bug bounty participation, malware analysis, and penetration testing**"*

With this statement established as part of my **Organizational Context** (GV.OC-01), I can use it to better understand my **risk appetite**, which is the amount of risk I'm willing to accept in pursuit of my goals. Already I can begin to visualize my unique threat landscape. I quickly drew the following diagram to help me brainstorm my **inherent risks**, the raw danger of compromise from some of my routine activities if I were to operate without any security controls.

![Activity Compromise Dangers](/assets/images/Activity_compromise_danger.png)

*Brainstorm of likelihood vs impact of common activities being compromised*

The thinking behind the placements on this graph include the fact that online banking or gaming are not particularly dangerous activities. Banking websites typically implement multi-factor authentication and have to continuously pass strict data security and privacy standards. Gaming on a PC often involves installing software that can make unexpected changes to my system but those dangers can be minimized somewhat by obtaining games from a reputable source. Of course the impact of a compromise of my banking activities would be devastating so care must still be taken to do this in a secure way. The most risky activity I identified would of course be malware research. Voluntarily handling and executing trojans, keyloggers, ransomware, and other malware carries high inherent risks that if ignored can be devastating not only to the pursuit of my goals but also financially and legally. For example, my devices could become part of a botnet that attacks other computers and breaks cybersecurity laws. 

Since my mission statement includes these riskier activities, I can't just avoid them. Instead I need to implement controls to reduce their danger and push those activities to the left, to an acceptable **residual risk** level. While my quick visualization gives a partial high level snapshot of some risks, I will create an official  **risk register** document to use as part of my **Risk Management Strategy** (GV.RM-06). This will help me to identify specific threats, calculate their severities, and associate them to proposed mitigating controls. For now, the document looks like this:

| Risk ID | Risk Description | Impact (1-5) | Probability (1-5) | Severity | Action | Mitigation | Added | Reviewed | Status |
| :--- | :--- | :---: | :---: | :---: | :--- | :--- | :--- | :--- | :--- |
| **RR-01** | Credentials stolen via phishing or infostealer | 5 | 2 | 10 | Mitigate | MFA, Password Manager, System Isolation | 13.03.2026 | | Open |
| **RR-02** | Accidental security research deletion or encryption by ransomware | 5 | 1 | 5 | Mitigate / Transfer | Local & Cloud Backups | 13.03.2026 | | Open |
| **RR-03** | IoT devices infected by botnet | 3 | 2 | 6 | Mitigate | Network Isolation, Monitoring | 13.03.2026 | | Open |
| **RR-04** | Loss of access to Google account | 5 | 1 | 5 | Mitigate | Remove single point of failure | 13.03.2026 | | Open |
| **RR-05** | Malware during research traverses to personal devices | 5 | 4 | 20 | Mitigate | System / Network Isolation, Firewall Rules, Monitoring | 13.03.2026 | | In progress |
| **RR-06** | Self-hosted service contains a backdoor / malware | 3 | 3 | 9 | Mitigate | Monitoring & Alerting | 13.03.2026 | | Open |
| **RR-07** | Sudden loss of power / internet | 3 | 1 | 3 | Accept | None | 13.03.2026 | | Closed |

*Risk Register*

By creating and continuously reviewing this document, I can see exactly what risks pose danger to my specific goals and track what is being done to treat those risks. In this case, I've chosen to use a simple 5-point scale for impact and probability which is then multiplied for severity. This helps me to prioritize risks with the highest severity. I have also evaluated probabilities assuming minimal or baseline controls while mitigation strategies are intended to reduce this to an acceptable residual risk level. As can be seen, I will reduce the risk of malware traversing to personal devices during research by implementing system / network isolation, firewall rules, and monitoring. While those actions will take time and add additional maintenance costs and complexity to my environment, the impact of that risk being realized is too great for me to ignore. On the other hand, I have also added the threat of the sudden loss of power or internet to the register but have chosen to simply accept that risk. This is because I am living in an area where losing power and internet access is rare and if it does happen I can just work in another location or use my phone as a hotspot in emergencies. 

A more interesting threat that I have identified is losing my account with Google. If that were to happen, I would lose access to bug bounty platforms, personal banking websites, penetration testing training resources, and a plethora of other services. Recently more and more people have reported account closures due to claimed breaches of conduct in their use of Google services. In many cases, the bans occurred automatically and without warning! While the evidence I have for this threat is primarily anecdotal, I cannot eliminate the possibility of this happening to me and the severe consequences it would have. For that reason, I have chosen to address that risk by removing this single point of failure and implementing a separation of duties when it comes to email and other 3rd party services. The treatment of this risk is also part of my **Supply Chain Risk Management** (GV.SC-07) that requires the identification of such third party service providers and considerations of the risks they introduce. Related to that is risk RR-06, which is about the threat of a self-hosted service that I use containing a backdoor or other malware. I will also be taking steps to treat that risk by implementing monitoring & alerting of when suspicions of this taking place arise. Of course, the risk register will be regularly reviewed to track progress and more risks may be added in the future.

Now when it comes to the subcategories under **Roles, Responsibilities, and Authorities** (GV.RR), it's important to note that in my case, I am the security researcher, system administrator, SOC analyst, finance officer, CISO, and other roles as the situation presents itself. While it might seem unnecessary, formalizing these roles will help me stay accountable and ensure that responsibilities of each are not overlooked. For instance, it will help to ensure that my activities analyzing malware samples do not outpace or supersede my administrative responsibilities to secure underlying operating systems, virtual environments, and networks. Additionally, my role as a finance officer in support of my cybersecurity program helps ensure that adequate resources are allocated in line with my risk strategies and policies  (GV.RR-03). A commonly used tool to clarify such roles and responsibilities is a RACI chart. It is often a simple spreadsheet that lists who is **R**esponsible, **A**ccountable, **C**onsulted, and **I**nformed for tasks that support a project or program. For me, the following RACI chart will serve more as a mental workflow to prevent conflicts of interest that may compromise my security.

| Activity | Security Researcher | System Administrator | SOC Analyst | Finance Officer | CISO |
| :--- | :---: | :---: | :---: | :---: |:---: | 
| Security monitoring & alert review | I | C | R | I | A |
| Operating system updates | I | R  | I | I |A |
| Network segmentation & firewall rules | I | R | I | I | A |
| Quarterly risk strategy review | C | I | C | I | R / A |
| Lab setup & hardening | R | C | I | I | A |
| Subscription & infrastructure budget | C | C | C | R | A

*RACI Chart: Roles & Responsibilities*

### Next steps

At this point, I have successfully used the **Govern** function to establish what my mission is, what the risks are, and clarify the roles and responsibilities needed to support my cybersecurity program. I effectively have a compass now calibrated to point me in the right direction. But to actually transform my security posture, I am going to need to know where I stand currently and exactly where I want to go. This is where **Organizational Profiles** will be used. In my case it's going to include the following:

 - **My Current Profile**: This will be an honest snapshot of my homelab environment. I will map my existing setup with the NIST CSF Core Subcategories to see where I am today.
 - **The Target Profile**: Based on the risks I've already identified, this will be the desired state of the security outcomes I want to achieve. 
 - **Gap Analysis**: This is where the action will take place. By comparing the profiles I can identify "gaps" and prioritize the steps needed to move closer to the target profile that I have defined. 

So stay tuned as in the next articles in this series we'll be diving deeper into the Identify, Protect, Detect, Respond, and Recover portions of CSF 2.0. I will even be putting back on my technical hat (think VLANs and SEIM) in some portions to actually get my homelab aligned with my new cybersecurity program.




