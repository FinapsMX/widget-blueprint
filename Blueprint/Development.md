This blueprint aims to provide an introduction for developers looking to create and maintain Mendix pluggable widgets. After following this guide, developers should be able to contribute towards the Finaps widget ecosystem in a standard way.

A standardized way of working confers the following benefits:
1. **Consistency:** Standardization ensures that all developers follow the same set of rules, practices, and guidelines when working on projects. This consistency leads to a more predictable and reliable development process.
2. **Quality Assurance:** By adhering to standardized processes and best practices, developers can produce higher-quality code. This reduces the likelihood of bugs, security vulnerabilities, and other issues that can arise from inconsistent or ad-hoc development approaches.
3. **Collaboration:** Standardization fosters better collaboration among team members. When everyone is on the same page regarding coding conventions, project structure, and development workflows, it's easier for team members to understand and work with each other's code.
4. **Onboarding and Training:** Standardization simplifies the onboarding process for new developers. They can quickly get up to speed by following established guidelines and practices. It also makes training more efficient, as there are clear references to follow.
5. **Maintainability:** Consistent coding standards and practices make code easier to maintain over time. When multiple developers are working on a project, they can understand and modify each other's code more effectively, reducing technical debt.

# Standards
1. **Typescript**: widgets should be written in Typescript as opposed to Javascript. TypeScript introduces static typing, allowing developers to define data types for variables, function parameters, and return values. This helps catch type-related errors during development, making code more robust and easier to maintain. Type annotations in TypeScript make the code more self-documenting. Developers can easily understand the expected types of variables and functions, leading to better code readability and fewer misunderstandings.
2. **Conventional Commits**: The Conventional Commits specification describes the way in which a developer should structure their commit messages. It provides an easy set of rules for creating an explicit commit history; which makes it easier to write automated tools on top of. 
	1. The format follows `<type>(<scope>): <message>`
		- `<type>` describes the purpose of the commit, such as "feat" for new features, "fix" for bug fixes, "docs" for documentation changes, and so on.
		- `<scope>` is optional and specifies the part of the codebase or component that the commit affects.
		- `<message>` provides a brief, yet descriptive, summary of the changes made in the commit.
3. **Semantic Versioning**: A set of rules for software developers to communicate changes to a software project in a clear and standardized way. It aims to convey meaning about the underlying changes in a version number.
	1. The format follows `Major.Minor.Patch` 
		1. `Major`: This number is incremented when there are incompatible changes that require users to make significant modifications to their code or when there are major new features. Incrementing the major version indicates that backward compatibility may be broken.
		2. `Minor`: This number is incremented when backward-compatible new features or enhancements are added to the software. It signifies that the changes do not break existing functionality and that users can safely update to this version.
		3. `Patch`: The patch version is incremented for backward-compatible bug fixes or minor improvements that do not introduce new features. These changes should not disrupt existing functionality.
4. **Quality assurance**: 
	1. **Static code analysis**: Analyzes the code without running it, looking for potential issues, code quality violations, and security vulnerabilities. This analysis checks for adherence to coding standards, identifies bugs, and provides suggestions for improvements
	2. **Vulnerability scans**: 
	3. **Build analysis**: Involves checking for errors, dependencies, and configuration issues during the build process. Build analysis helps ensure that the software can be successfully compiled and run, reducing potential issues in the final product.
	4. **Automated testing**: Using tools to automatically execute test cases and verify the behavior of a program or application. It helps identify defects, ensure code quality, and maintain software reliability by running tests automatically, allowing for frequent and consistent evaluation of software functionality and performance.
5. **Branch naming**: All development happens on a branch created from the `main` branch. Naming convention follows `<type>/<description`.
	1. `<type>`: `[feature/fix/infrastructure/debt]`

# Getting started
Now that you are familiar with the standards and practices of widget development, you can begin developing. Check out the [[Blueprint]] for a getting started guide.