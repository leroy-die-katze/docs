# Workplace Safety - Debris Detection

![debris-detection](../../images/debris-detection.png)

A manufacturer wants to deploy a computer vision system that monitor workspaces to ensure no tools, spare parts or debris are left on the factory floor. This sample walks through the entire journey of the project and illustrates how IaC combines generative AI and classic software lifecycle management to evolve a simple prototype into a full-scale, enterprise-grade production system. 

## Prototype
In the prototype phase, the requirement is expressed as “A computer vision system that can detect screwdrivers on the factory floor and send a warning message to an email address”.

Based on the requirement expressed in natural language, the generative AI generates a codified intention as show in the following YAML sample:
```yaml
intention:
  inputs:
  -"camera frame"
  capabilities:
  -"generate warning message"
    constraints:
    -"screwdrivers in camera frame"
  outputs:
  -"warning message"
    method: email
    destination: $param(admin_email)
```
This codified intention can be used in several ways:
 1.	The codified intention is fed to IaC code generation that uses a discovery process to discover capability vendors who can fulfill the intention. Once the vendor is selected, a software package/update can be generated and automatically deployed using a CI/CD pipeline.
 2.	If the vendor discovery process fails, a work item is generated in engineering tracking system to get a capability vendor implemented.
 3.	The codified intention can also be used as a requirement document for traditional software development process. 

 Let’s say, the team chose to use a Raspberry Pi with a USB camera, and trained a simple AI model that can detect screwdrivers at roughly 4 frames per second. Because we don’t have more captured requirements, the prototype is completely successful.

 > **NOTE**: As you may have noticed, lots of fields in IaC use natural language descriptions instead of strongly typed data structures. This is an important design choice of IaC that separates it from common code generation systems: it focuses on abstract capability matching instead of service invocation details, which can be handled by existing code generation or automation systems. And when IaC does capability matching, it uses AI to match vendor descriptive vectors with constraints. Hence IaC doesn’t need a rigid data schema for its operation.

## A reality check - view of the camera

It turned out, the team missed an important factor: the camera won’t be scanning the floor taking close-up shots. Instead, it will be mounted high up monitoring an entire segment of the factory floor. When such requirement is clarified, the AI model discovers that the codified intention doesn’t offer enough details to answer the question “would it work with a wide-angle shot of the entire factory floor?”. Then, AI generates an updated intention with additional details:
```yaml
intention:
  inputs:
  -"camera frame"
    contraints:
      location: fixed
      view: "wide angle"
  capabilities:
  -"generate warning message"
    constraints:
    -"screwdrivers in camera frame"
  outputs:
  -"warning message"
    method: email
    destination: $param(admin_email)
```
This renewed intention triggers either a new vendor to be selected, or additional work items to be generated for the engineering team to implement a qualified vendor – that does screwdriver detection in a wide-angle picture instead of a closeup picture. A bunch of issues need to be resolved: first, camera resolution isn’t high enough to capture enough details from far away; second, the AI model isn’t trained with correct samples; third, the new AI model becomes too complex for the current hardware profile and beefier hardware with GPU is required. 

The engineering team worked hard, chose a different hardware, trained a new AI model. At the same time, they annotated their vendor registration with additional details:
> **NOTE**: note the root is ```vendor``` instead of ```intention``` here.
```yaml
vendor:
  inputs:
  -"camera frame"
    contraints:
      location: fixed
      view: "wide angle"
  capabilities:
  -"generate warning message"
    constraints:
    -"screwdrivers in camera frame"
    requirements:
      GPU: "Nvdia Jetson"
      memory: 32GB
  outputs:
  -"warning message"
    method: email
    destination: $param(admin_email)
```
## A classic touch - what about HA?

One benefit of IaC is that you don’t have to always involves AI when refining your system. You can directly review the IaC artifact and add new annotations to reflect your requirements. 

In this case, an architect reviewed the IaC artifact and asked, “what about HA?” Unfortunately, HA could be interpreted differently: HA may means failing over camera stream processing to a different compute device, or a redundant infrastructure of cameras and compute devices, or a battery-based backup system that can be used during power outages, and so on. 

Let’s say the architect added an anotation without any details:
```yaml
constraints:
  HA: enabled
```
This disqualifies the currently selected vendor and triggers a new vendor selection or new development work items.

As the new round of work progresses, the IaC is evolved, reviewed, and aligned among the teams, with decisions captured in the artifact, such as:
```yaml
constraints:
  HA: "duplicated infrastructure"
    topology: "primary-secondary"
    switch: automatic
    trigger: "crash or disconnect"
```
## Rolling out - vendor profile
As the developers works on the vendor, they also annotate their vendor profile with details of how to deploy the vendors. IaC doesn’t aim to provide a new application model or deployment artifact format. Instead, the vendor profile should contain information about how to acquire and apply the required (Infrastructure as Code) artifact to deploy the vendor. For example, if the vendor is deployed with a Helm chart, then the vendor profile may look like:

```yaml
vendor:
  …
  deployment:
    artifact: Helm
      repo: oci://contoso.com/helm
      chart: my-camera
      version: 0.3.1
```
This profile can be communicated with the operational team to do a proper vendor deployment. At the same time, the profile of the selected vendor is also fed back to the AI model so that AI is equipped with knowledge to answer deployment questions like “which version of debris detection is running on floor 3?”

## A futuristic look 
As IaC grows, two things are expected: first, an ecosystem of vendors will grow and offers a broad range of quality, out-of-box vendor choices. These vendors can host services, software packages, or even professional services, with both functional and non-functional annotations such as SLA, cost, locality, encryption level, and many more. IaC will use a multi-vector model to select vendors based on both vendor qualifications and project constraints.

Second, with customer’s consensus, as IaC sees more projects, it builds up the knowledge to ask the correct questions even before human users realize them, such as “how the system tolerate dust?”, “does it work under low lighting conditions”, “how does it react to objects in motion?”, etc.

## Summary
This simple sample shows how IaC unifies generative AI process and classic software engineering process into a unified, interactive workflow. While you leverage the full power of AI, you don’t lose control as you can explicit describe and verify the exact required capabilities, and required vendor qualifications to fulfill user’s intention.  