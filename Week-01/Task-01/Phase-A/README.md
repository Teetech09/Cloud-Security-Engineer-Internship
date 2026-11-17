Task 1 – AWS Shared Responsibility Model

Objective

To understand the AWS Shared Responsibility Model and how security responsibilities differ between on-premises environments, IaaS, PaaS, and SaaS cloud service models.

1. AWS Shared Responsibility Model

I reviewed the official AWS Shared Responsibility Model documentation to understand how security responsibilities are divided between AWS and the customer.

The main concept I learned is that moving to the cloud does not mean that AWS takes care of every aspect of security. AWS is responsible for securing the infrastructure that runs its cloud services, while customers remain responsible for securing the parts of the environment that they control. The exact responsibilities depend on the AWS service being used.

2. IaaS, PaaS and SaaS Responsibilities

In IaaS, the cloud provider manages the physical infrastructure, while the customer has more responsibility for the operating system, applications, configurations, and data.

In PaaS, the provider manages more of the underlying infrastructure and platform, allowing the customer to focus mainly on their applications, configurations, and data.

In SaaS, the provider manages most of the technology stack and the customer mainly manages how the service is configured, who can access it, and how their data is used and protected.

The key lesson I learned is that customer responsibility generally becomes smaller as the service moves from IaaS towards SaaS, but the customer still has security responsibilities in every model.

3. On-Premises vs Cloud Analogy

I compared the different responsibility models to managing a home.

With an on-premises environment, it is like owning and maintaining an entire house. The owner is responsible for the building, electricity, plumbing, security, and everything inside it.

With IaaS, it is more like renting a basic building where the provider takes care of the structure, while the customer is responsible for much of what is installed and operated inside.

With PaaS, the provider manages more of the building and its facilities, while the customer focuses on what they build and operate on the platform.

With SaaS, it is similar to using a fully managed service where the provider handles most of the underlying environment, and the customer mainly manages their account, settings, users, and data.

4. Shared Responsibility Diagram

I created a diagram using Draw.io to show which layers are primarily managed by AWS and which are managed by the customer across on-premises, IaaS, PaaS, and SaaS environments.

Diagram: "shared-responsibility-diagram.png"

5. Tools Used

- AWS Shared Responsibility Model documentation
- ChatGPT
- Draw.io

6. Key Learning

The main lesson from this task is that cloud security is a shared responsibility. The cloud provider secures the infrastructure it operates, while the customer is responsible for securing the resources, configurations, identities, applications, and data that are within their control.

Understanding where the responsibility lies is important because a security issue can occur when a customer assumes that the cloud provider is responsible for something that actually remains the customer's responsibility.

7. Completion Checklist

- [x] Read the official AWS Shared Responsibility Model documentation
- [x] Summarized IaaS, PaaS, and SaaS responsibilities in my own words
- [x] Created an analogy comparing on-premises and cloud responsibilities
- [x] Created a shared responsibility diagram using Draw.io

Status

Completed
