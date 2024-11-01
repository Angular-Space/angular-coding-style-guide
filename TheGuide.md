# Angular Style Guide

### Introduction
**📜 Guideline**: Remember, these are guidelines meant to help you and your team find the best workflow. Always assess your team’s experience, project goals, and productivity needs rather than strictly adhering to rigid rules.

**💡 Why We Recommend This**  
Rigid rules can limit productivity and overlook the unique needs of a team. Prioritising developer experience keeps the team motivated and effective. Most applications don’t need to be hyper-optimised from day one; focus on building features over strictly enforcing coding rules.

---

### Testing for Longevity

**🧪 Guideline**: Testing is essential for adapting to future changes. Invest in tests to create a foundation that supports long-term scalability.

**💡 Why We Recommend This**  
Testing ensures that changes and new features can be integrated without breaking existing functionality. A robust test suite reduces future maintenance and creates a stable codebase that adapts well over time.

**✅ When to Use Testing**  
- For critical logic that directly impacts business functionality.
- For reusable components or services that may need to evolve with the project.

**🚫 When to Avoid Over-Testing**  
- Avoid testing trivial UI elements or components with minimal logic, as this can increase maintenance without providing meaningful value.

---

### Design Language System

**🎨 Guideline**: Code should adhere to a design language system (DLS). Handle breaking changes using semantic versioning and migrations.

**💡 Why We Recommend This**  
A DLS creates visual and functional consistency across the application, making it easier to maintain and scale. Semantic versioning and migration support ensure that changes don’t inadvertently disrupt dependent parts of the application.

**✅ When to Use DLS-Based Coding**  
- When creating UI components shared across different parts of the application.
- When building reusable modules that need to align with company branding and standards.

**🚫 When to Avoid Rigidity in DLS**  
- For prototype features where rapid iteration is prioritised over strict adherence to DLS.

---

### Company Schematics

**🛠️ Guideline**: Use company-defined schematics to standardise component, service, and directive creation.

**💡 Why We Recommend This**  
Schematics standardise the structure and reduce setup time, ensuring everyone on the team follows the same conventions. This consistency improves maintainability and reduces onboarding time for new team members.

**✅ When to Use Company Schematics**  
- When creating new components, services, or directives that should align with team standards.
- For larger projects where consistency is key to managing a scalable codebase.

**🚫 When to Avoid Custom Schematics**  
- For small, unique projects where the schematics may add unnecessary overhead.

---

**🏁 End of Angular Style Guide**
