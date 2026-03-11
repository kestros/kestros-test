## FAQ

**Q: What is this repository for?**
A: This repository is used for end-to-end agent pipeline validation in the Kestros development system. It exists to test the full workflow from task creation through PR review without affecting production code.

**Q: Why are the commits in this repo not real features?**
A: kestros-test is a synthetic test bed. The changes merged here are intentionally minimal — they exercise the agent lifecycle (BA scoping, Lead Dev guidelines, Developer implementation, QA review, Danny approval) rather than delivering functional software.

**Q: How does this repo relate to Kestros CMS?**
A: kestros-test has no runtime dependency on Kestros CMS. It exists solely as a safe target for agent pipeline tests so that real production repositories are not affected during system validation.
