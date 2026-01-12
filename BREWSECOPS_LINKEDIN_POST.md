# LinkedIn Post — BrewSecOps Project

---

## Version 1: Technical Deep-Dive (For Engineering Managers)

---

**I built a production-grade DevSecOps platform from scratch. Here's what 84 commits and 76 AWS resources taught me.**

After mass layoffs hit the tech industry, I decided to stop waiting and start building.

**The result:** BrewSecOps — a fully deployed, multi-environment AWS infrastructure running a containerized 3-tier application.

**Live:** https://dev.brewsecops.online

---

**What I built:**

→ **Infrastructure:** 76 AWS resources across 11 reusable Terraform modules
→ **Environments:** Dev, Staging, Production (code-ready, cost-optimized)
→ **Security:** 6-layer scanning pipeline + AWS WAF with 6 custom rules
→ **Pipeline:** 6-minute CI/CD with automated security gates
→ **Architecture:** Multi-AZ ECS Fargate in Frankfurt (eu-central-1)

---

**Key decisions I'm proud of:**

1. **Dual dependency scanning** — OWASP (government-backed NVD) + Snyk (proprietary intel). Different databases catch different vulnerabilities.

2. **Supply chain security** — Cosign keyless signing + SBOM generation. Every container is cryptographically verified.

3. **Cost optimization** — Only dev deployed during job search. Staging/prod code-ready but paused. **$550/month saved** while maintaining full demonstration capability.

4. **Multi-AZ from day one** — Not an afterthought. High availability baked into the architecture.

---

**The numbers:**

```
• 2,767 lines of Terraform
• 7,401 lines of application code
• 6-minute pipeline (3 stages)
• 77 screenshots documenting everything
• $226/month running cost (dev)
• 0 critical vulnerabilities
```

---

**Tech stack:**

React 18 • Node.js 20 • PostgreSQL 15 • ECS Fargate • Terraform • GitHub Actions • AWS WAF • CloudWatch • Cosign • SonarCloud • Trivy

---

**What's next:**

Currently seeking Senior DevOps/Platform Engineer roles in Berlin or remote (Germany).

EU work authorization. Available immediately.

If your team values security-first infrastructure and engineers who ship — let's talk.

---

GitHub: github.com/AkingbadeOmosebi/brewsecops

#DevOps #AWS #Terraform #CloudEngineering #DevSecOps #Berlin #OpenToWork

---

---

## Version 2: Shorter & Punchier (For Broader Reach)

---

**I deployed 76 AWS resources to prove I can do the job. Here's the project.**

BrewSecOps — A production-grade DevSecOps platform I built from scratch.

**Live:** https://dev.brewsecops.online

---

**The highlights:**

→ Multi-environment infrastructure (Dev/Staging/Prod)
→ 11 reusable Terraform modules
→ 6-minute CI/CD pipeline with security scanning
→ AWS WAF with 6 custom rules
→ Multi-AZ ECS Fargate (Frankfurt)
→ Supply chain security (Cosign + SBOM)
→ 77 screenshots documenting everything

---

**Why I built it:**

The job market is tough. So I stopped applying and started shipping.

This isn't a tutorial project. It's:
• 84 commits of real iteration
• Real debugging challenges (documented)
• Real cost optimization ($550/month saved)
• Real security implementation

---

**Currently seeking:** Senior DevOps/Platform Engineer roles in Berlin or remote.

EU work authorization. Available now.

GitHub: github.com/AkingbadeOmosebi/brewsecops

#OpenToWork #DevOps #AWS #CloudEngineering #Berlin #Terraform

---

---

## Version 3: Story-Driven (For Maximum Engagement)

---

**3 months ago, I got laid off.**

Instead of panicking, I asked myself: *What would prove I'm ready for a senior role?*

The answer: Build something real. Deploy it. Document everything.

---

**Today, I'm sharing BrewSecOps.**

A production-grade AWS platform I designed, built, and deployed myself.

**Live:** https://dev.brewsecops.online

---

**What makes it different from typical portfolio projects:**

**It's actually deployed.**
76 AWS resources. Multi-AZ. Frankfurt region. Running right now.

**It's actually secure.**
6 scanning tools in the pipeline. WAF with custom rules. Supply chain signing.

**It's actually documented.**
77 screenshots. Architecture docs. A "Challenges & Learnings" file with real debugging stories.

**It's actually cost-conscious.**
$226/month for dev. Staging and prod are code-ready but paused. Because infrastructure costs money, and I respect that.

---

**The stack:**

Terraform (11 modules) • ECS Fargate • GitHub Actions • AWS WAF • CloudWatch • Cosign • React • Node.js • PostgreSQL

---

**The metrics:**

• 84 commits
• 2,767 lines of Terraform
• 6-minute pipeline
• 0 critical vulnerabilities

---

**I'm not looking for sympathy. I'm looking for opportunity.**

Senior DevOps / Platform Engineer roles.
Berlin or remote (Germany).
EU work authorization. Available immediately.

If you're hiring engineers who ship — my GitHub speaks for itself.

github.com/AkingbadeOmosebi/brewsecops

---

#OpenToWork #DevOps #AWS #Terraform #CloudEngineering #Berlin #TechLayoffs #Hiring

---

---

## Posting Tips

| Tip | Why |
|-----|-----|
| Post Tuesday-Thursday, 8-10 AM CET | Peak LinkedIn engagement for German market |
| Add 3-5 images | Architecture diagram, pipeline screenshot, live app, WAF dashboard |
| Tag companies you want to work for | Increases visibility to their recruiters |
| Engage with comments within 1 hour | Algorithm boost |
| Use Version 3 for maximum reach | Story-driven posts get 2-3x engagement |

---

## Suggested Images to Attach

1. Architecture diagram (if you have one)
2. Screenshot of live app (https://dev.brewsecops.online)
3. GitHub Actions pipeline (green checkmarks)
4. AWS WAF dashboard showing rules
5. Terraform modules folder structure

---
