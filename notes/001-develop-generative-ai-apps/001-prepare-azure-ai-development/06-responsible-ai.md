# 06 Responsible AI

[Responsible AI](https://learn.microsoft.com/en-us/training/modules/prepare-azure-ai-development/6-responsible-ai)

Models are probabilistic and depend on training data. Users trust the human-like output, so a wrong or biased result can harm people. Microsoft's six principles:

- **Fairness.** Treat people fairly. A loan model must not use gender, ethnicity, or similar factors. Check that training data represents the people affected, and measure accuracy on subgroups. Tools that score bias are not enough on their own.
- **Reliability and safety.** The system has to behave as expected. Examples: an autonomous vehicle, or a model that recommends a prescription. Test and control releases. Set a threshold on the confidence score before acting on a prediction.
- **Privacy and security.** Training data and live inputs can contain personal data. Protect that data and customer content in training and in production.
- **Inclusiveness.** The benefit should reach people regardless of ability, gender, sexual orientation, ethnicity, or similar factors. Include a diverse group in design, development, and testing.
- **Transparency.** Tell users the purpose, how it works, and the limits. For a model, that includes how many cases it was trained on, which features drive predictions, and the confidence score. For facial recognition, say how images are used, how long they are kept, and who can access them.
- **Accountability.** People who train the model and write the decision logic are responsible, even when the system looks autonomous. Work inside a defined governance and legal framework.
