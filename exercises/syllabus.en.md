
# GitHub Advanced Security for developers workshop

## Syllabus

### Day 1

#### 1. What is GitHub Advanced Security? 
What is GitHub's approach to the current software security challenges the industry is facing. Overview of what are GitHub Advanced Security consists of and how does it fit in the overall GitHub as platform.

Intros - 5 mins
Presentation - 10 mins

**Time: 15 minutes**

#### 2. GitHub Advanced Security Licensing
In this section we look into the GHAS licensing model and discuss possible license consumption situation with allocations and releasing of licenses that GHAS users show be aware of.


Presentation - 5 mins
Demo 1 - Show current license usage/ what is the active committer count/ what is the license allocation. (5 minutes)

**Time: 10 minutes**

#### 3. GHAS Enablement
Learn how do you manage GHAS policy on an enterprise level. Learn how GHAS enablement on organization and repository level means. Go over the available out of the box options for enablement and tracking coverage.
Conclude with the approach that NBS is taking to making GHAS available and activated to development teams/repos and demonstrate the short-/mid-term IssueOps solutions that will be serving Development teams to onboard and get started with GHAS 

Presentation - 10 mins
Demo 2 - Control GHAS Enterprise Level policy (5 mins)
Demo 3 - Onboarding process at NBS issueOps (5 mins)
Exercise 1 - Enabling GHAS on your repository (10 minutes)

**Time: 30 minutes

Feedback:
- Show whats coming out and how it works
- IssueOps just a demo from NBS side. Short to medium term solution;


#### 4. How does it work?

Explain the suggested design of most scans and security checks happen in PR and how does that fit into the developer workflow. Discuss the reasons, and why the benefits outweigh the pitfalls of this approach.

Go over each of the GHAS products and explain how does it work:
- Dependancy Graph, Dependabot, Dependancy Review. We will get familiar with the available features and products but we will not go too much into the depths of them. 

- Secret Scanning and Secret Scanning Custom Patterns. We will learn how secret scanning works under the hood and what you as Ops Engineer can and cannot influence or control. We will practically also get familiar with writing custom patterns and learn how to debug them.

- Code Scanning, CodeQL CLI, CodeQL in CI. We will dig into what the NBS implementation in CBJ and GitHub Actions of CodeQL scans will be. We will practically setup GitHub Actions to analyze a multi-language project and understand what complexities and challenges such situations might bring.

Presentation - 20 minutes
Optional Exercise - Bring GHAS to your IDE (5 mins)
Exercise 2 - Dependabot (10 minutes)
Exercise 3 - Dependancy Review (15 mins)
Exercise 4 - Secret Scanning and Push Protection (10 mins)
Exercise 5 - Secret Scanning - Custom Secret Scanning Pattern + Push protection (15 mins)

**Time: 80 minutes**

---

### Day 2


#### 4. How does it work? (Contd.)

Explain the suggested design of most scans and security checks happen in PR and how does that fit into the developer workflow. Discuss the reasons, and why the benefits outweigh the pitfalls of this approach.

Go over each of the GHAS products and explain how does it work:
- Dependancy Graph, Dependabot, Dependancy Review. We will get familiar with the available features and products but we will not go too much into the depths of them. 

- Secret Scanning and Secret Scanning Custom Patterns. We will learn how secret scanning works under the hood and what you as Ops Engineer can and cannot influence or control. We will practically also get familiar with writing custom patterns and learn how to debug them.

- Code Scanning, CodeQL CLI, CodeQL in CI. We will dig into what the NBS implementation in CBJ and GitHub Actions of CodeQL scans will be. We will practically setup GitHub Actions to analyze a multi-language project and understand what complexities and challenges such situations might bring.


Presentation - 10 minutes
Exercise 6 - Code Scanning and CodeQL (15 mins)
Exercise 7 - CodeQL config customization and custom query (15 mins)
Extra - CodeQL workflow optimization (10 mins)

**Time: 45 minutes**

#### 5. Access, Notifications and alerts

Look into what access requirements to the security features and GHAs alerts including how do you manage this. Understand how this the requirements map to the the standard GitHub pre-defined roles: Read, Triage, Write, Maintain and Admin, but also, the special Security Manager user role. We will exercise the possibilities for creating custom roles for Security Champions as per predefined requirements. 

Further, we will investigate what notifications are sent out by the GHAS products and to whom and understand what comes by default vs what could people further configure.
Lastly, we will discuss some considerations around notifications and how to avoid situations where people get flooded and essentially notifications hove the counter effect to users and how they can manage that. 

Presentation - 10 mins
Demo 3 - GHAS notifications and personal configurations (5 mins)
Demo 4 - Default user roles in Security Features context; Defining a custom role; Security manager role (10 mins)

**Time: 25 minutes**

#### 6. Integrations - GHAS API and Webhooks

Learn about ways to interact with GHAS outside of what is coming of the box through the available audit log, GHAS APIs and webhooks. 

We will look in detail what the GHAS APIs and Webhook provide around GHAS and what GHAS events are logged into the audit log. We will focus on a common use case to integrate other security tools under GHAS and Code Scanning with a goal to make Code Scanning one central spot where developers can find all there (static) security analysis results. 

Additionally, we will also integrate another PR check based on GHAS secret scanning alerts by leveraging the APIs


Presentation - 20 minutes
Exercise 8 - Run tool Checkov in PR and integrate in Code Scanning (15 mins)
Exercise 9 - Introduce Secret Scanning Review action in PR (10 minutes)

**Time: 75 minutes**


#### 7. Troubleshooting GHAS

In this last part of the workshop we will look at most common issues with GHAS adoption. We will start with general service health-checks you could do but focus on Code Scanning and CodeQL. Go over what are common problems that you might expect around CodeQL runs and configurations and how you go about troubleshooting them. 
We will look at what a CodeQL debug package contains and how you can use that to triage the problem.

Presentation - 15 mins
Exercise 10 - Generate CodeQL debug and identify a problem (15 mins)

**Time: 50 minutes**
