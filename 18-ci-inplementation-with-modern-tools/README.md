# Understanding the wholistic CI inplementation with modern tools

### Experience Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

**IMPORTANT NOTICE** This project has some initial theoretical concepts that must be well understood before moving
on to the practical part. Please read it carefully as many times as needed to completely digest the most important and fundamental DevOps concepts. To successfully implement this project, it is crucial to grasp the importance of the entire `CI/CD` process, roles of each tool and success metrics-so we encourage you to thoroughly study the following theory until you feel comfortable explaining all the concepts (e.g., to your new junior colleague or during a job interview).
In previous projects, you have been deploying a tooling website directly into the `/var/www/htel` directory on dev servers. Well, even though that worked fine, and we were able to access the website, it is not the best way to do it. Real world web application code written on `Java`, `.NET` or other compiled programming languages require a build stage to create an executable file. The executable file (e.g., jar file in case of Java) contains all the codes embedded, and the necessary library dependencies, which the application needs to run and work successfully.
Some other programs written languages such as PHP, JavaScript or Python work directly without being built into an executable file-these languages are called interpreted. That is why we could easily deploy the entire code from git into `var/www/html` and immediately the webserver was able to render the pages in a browser. However, it is not optimal to download code directly from it onto our servers. There is a smarter way to package the entire application code, and track release versions. We can package the entire code and all its dependencies into an archive such as .tar.gz or
zip, so that it can be easily unpacked on a respective environment's servers.
For a better understanding of the difference between compiled vs interpreted programming languages - read this short article.
In this project, you will understand and get hands on experience around the entire concept around CI/CD from applications perspective. To fully gain real expertise around this idea, it is best to see it in action across different programming languages and from the platform perspective too. From the application perspective, we will be focusing on PHP here; there are more projects ahead that are based on Java, Node.js, .Net and Python. By the time you start working on Terraform, Docker and Kubernetes projects, you will get to see the platform perspective of CI/CD in action.

## What is continous integration

Continuous integration (CI) is a software development practice that requires developers to frequently merge their code changes into a shared mainline, often several times a day. This practice is designed to identify and resolve code conflicts early in the development process, rather than waiting until the software is about to be released.

CI has several benefits, including:

+ **Early detection of errors:** CI helps to identify and resolve code conflicts early in the development process, which can save time and effort in the long run.

+ **Improved code quality:** CI helps to ensure that code is of high quality by automating the testing process.

+ **Reduced risk of deployments:** CI helps to reduce the risk of deployments by ensuring that code is stable and reliable

+ **Increased developer productivity:** CI can help to increase developer productivity by automating the build and testing process.

## continous intregration in the real world

In the real world, CI is implemented by organizations of all sizes, from small startups to large enterprises. It has become an indispensable tool for delivering high-quality software quickly and reliably.

# Devops terminology and success metrics

**Version control** is the stage where developers get commited and pushed testing locally

**Build** is the process of converting source code into a standalone software artifact that can be run on a computer

**Unit-test** is a small, self-contained test that is designed to test a single unit of code, such as a function or class

**Deploy** is the process of making a software application or update available to users in a production environment

**Auto-test** In software development, automated testing (also known as test automation) is the use of software to execute tests, typically scripts written in a programming language. Automated testing can be used to test a wide range of software components, including code, systems, and end-to-end (E2E) tests

**Deploying to production** is the final step in the software development lifecycle, where a new or updated version of an application is released to the target environment (production) where it is intended to be used by end-users

**measure and validate** measures the time it takes for a change to go from code commit to production deployment.

## common best practice of ci/cd

+ Maintain a code repository
+ Automate build process
+ Make builds self-tested
+ Everyone commits to the baseline every day
+ Every commit to baseline should be built
+ Every bug-fix commit should come with a test case
+ Keep the build fast
+ Test in a clone of production environment
+ Make it easy to get the latest deliverables
+ Everyone can see the results of the latest build
+ Automate deployment (if you are confident enough in your CI/CD pipeline and willing to go for a fully automated Continuous Deployment)