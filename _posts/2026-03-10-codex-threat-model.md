---
title: "Codex Threat Modeling Skill"
date: 2026-03-10 09:30:00 +0100
categories: [security, ai, appsec]
tags: [codex, threat-modeling, ai, code-review]
---

## Security Threat Modeling with Codex

While exploring OpenAI's Codex I noticed an interesting skill called "Security Threat Model". 

![codex1](/assets/img/posts/2026-03-10-codex-threat-model/codex-threatmodel1.png)

Skills can be thought about as: A packaged set of instructions + tools that teach the AI how to do a task. The structure below is what a skill looks like practically.

```bash
skill-name/
 ├─ skill.md        (instructions for the AI)
 ├─ scripts/        (optional code)
 ├─ resources/      (checklists, templates)
 └─ metadata.json
```

While the skill itself has prompting that would satisfy a threat model, I used this for my testing purposes.

```text
Use the security-threat-model skill on this repository.

Steps:
1. Infer system architecture from code and configuration.
2. Identify system components, services, and dependencies.
3. Generate a Mermaid data flow diagram.
4. Identify trust boundaries and sensitive assets.
5. Enumerate entry points and untrusted inputs.
6. Produce a STRIDE threat model mapped to components and data flows.
7. Recommend mitigations.

Focus on:
- authentication
- secrets management
- data exposure
- injection risks
- third-party integrations.
```

## Quick Test

For a quick test I decided on using the popular OWASP Juice Shop as a codebase for the
skill to assess. While using the desktop version of codex for my initial test, I decided to use the CLI version for this blog entry to see if there were any noticeable difference in the outcomes.

![codex2](/assets/img/posts/2026-03-10-codex-threat-model/codex-threatmodel2.png)

![codex3](/assets/img/posts/2026-03-10-codex-threat-model/codex-threatmodel3.png)

The prompted desktop codex version seemed more detailed, netherless it still produced a detailed threat-model.

{% include embed/youtube.html id='S-nekP_-L5M' autoplay=true mute=true %}

I've already started using this as a learning opporunity and threat-modeling both things in my personal life as well as for technology at work. For instance, this is a threat model of the SaaS platform Tray.ai. Codex took public documentation and knowledge of LCNC platforms and created what I consider a very accurate threat model.


![codex4](/assets/img/posts/2026-03-10-codex-threat-model/codex-threatmodel4.png)


# References
- https://developers.openai.com/api/docs/guides/tools-skills/
- https://developers.openai.com/codex/security/threat-model/