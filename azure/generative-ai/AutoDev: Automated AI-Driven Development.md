AutoDev: Automated AI-Driven Development
========================================

Michele Tufano Microsoft Redmond, USA

Anisha Agarwal Microsoft Redmond, USA

Jinu Jang Microsoft Redmond, USA

Roshanak Zilouchian

Neel Sundaresan

Moghaddam

Microsoft

Microsoft

Redmond, USA

Redmond, USA

Abstract
========

The landscape of software development has witnessed a paradigm shift with the advent of AI-powered assistants, exemplified by GitHub Copilot. However, existing solutions are not leveraging all the potential capabilities available in an IDE such as building, testing, executing code, git opera- tions, etc. Therefore, they are constrained by their limited capabilities, primarily focusing on suggesting code snippets and file manipulation within a chat-based interface.

To fill this gap, we present AutoDev, a fully automated AI-driven software development framework, designed for autonomous planning and execution of intricate software engineering tasks. AutoDev enables users to define complex software engineering objectives, which are assigned to Au- toDev's autonomous AI Agents to achieve. These AI agents can perform diverse operations on a codebase, including file editing, retrieval, build processes, execution, testing, and git operations. They also have access to files, compiler out- put, build and testing logs, static analysis tools, and more. This enables the AI Agents to execute tasks in a fully au- tomated manner with a comprehensive understanding of the contextual information required. Furthermore, AutoDev establishes a secure development environment by confining all operations within Docker containers. This framework incorporates guardrails to ensure user privacy and file secu- rity, allowing users to define specific permitted or restricted commands and operations within AutoDev.

In our evaluation, we tested AutoDev on the HumanEval dataset, obtaining promising results with 91.5% and 87.8% of Pass@1 for code generation and test generation respectively, demonstrating its effectiveness in automating software engi- neering tasks while maintaining a secure and user-controlled development environment.

1 Introduction
==============

As developers increasingly adopt AI assistants such as Chat- GPT for their development tasks, productivity gains become evident. AI coding assistants have further advanced into integrated development environments (IDEs) like GitHub Copilot \[2\], where they offer code suggestions both within chat interfaces and directly within files.

<figure>

![](figures/0)

<!-- FigureContent="Conversation Test this method User Eval Environment write <file> <content> Run test AI Agent Actions (1) X Error <failure log> OpenAI retrieval signature GPT-4 Repository Content result write <file> <fix> Run test Success" --> <figcaption>

Figure 1. AutoDev enables an AI Agent to achieve a given objective by performing several actions within the repository. The Eval Environment executes the suggested operations, providing the AI Agent with the resulting outcome. In the conversation, purple messages are from the AI agent, while blue messages are responses from the Eval Environment.

</figcaption> </figure>

However, these AI coding assistants, despite their integra- tion into IDEs, exhibit limited functionalities and lack con- textual awareness\[3, 6\]. They often do not leverage all IDE capabilities such as invoking linters, compilers, or executing command-line operations, and consequently, developers still need to manually validate syntax and ensure the correctness of AI-generated code, execute the codebase, and inspect error logs.

AutoDev bridges this gap by offering autonomous AI agents the ability to execute actions such as file editing, re- trieval, build, testing, and CLI commands directly within the repository to achieve user-defined objectives, thus enabling the completion of complex tasks autonomously.

AutoDev offers the following key features: (i) the ability to track and manage user and AI agents conversations through a Conversation Manager, (ii) a library of customized Tools to accomplish a variety of code and SE related objectives, (iii) the ability to schedule various AI agents to work col- laboratively towards a common objective through an Agent Scheduler, and (iv) the ability to execute code and run tests through an Evaluation Environment.

arXiv:2403.08299v1 \[cs.SE\] 13 Mar 2024 :selected: :unselected: :unselected: :selected: :unselected: :unselected: :selected: :selected: :selected:

<!-- PageHeader="Michele Tufano, et al." -->

Figure 1 illustrates a high-level example of the AutoDev workflow. The user defines an objective (e.g., testing a spe- cific method). The AI Agent writes tests in a new file and initiates the test execution command, all within a secure Evaluation Environment. The output of the test execution, including failure logs, is then incorporated into the conver- sation. The AI agent analyzes this output, triggers a retrieval command, incorporates the retrieved information by edit- ing the file, and re-invokes the test execution. Finally, the environment provides feedback on the success of the test execution and completion of the user's objective.

The entire process is orchestrated by AutoDev autonomously, requiring no developer intervention beyond setting the ini- tial objective. In contrast, with existing AI coding assistants integrated into IDEs, developers would have to manually execute tests (e.g., run pytest), provide failure logs to the AI chat interface, possibly identify additional contextual infor- mation to be incorporated, and repeat validation actions to ensure test success after the AI generates revised code.

AutoDev draws inspiration from previous work in the area of autonomous AI agents. For example, AutoGen \[22\] is a framework which orchestrates language model work- flows and facilitates conversations between multiple agents. AutoDev extends AutoGen by going beyond conversation management and enabling agents to directly interact with the code repository, executing commands and actions au- tonomously. Similarly, AutoDev builds upon Auto-GPT \[8\], an open-source AI agent for autonomous task execution by offering code and IDE specific capabilities to enable execu- tion of complex software engineering tasks.

In our evaluation, we assess the capabilities of AutoDev us- ing the HumanEval dataset\[5\], originally designed for code generation from natural language descriptions (docstrings). Additionally, we extend the evaluation to include the test case generation task, showcasing the versatility of AutoDev in handling diverse software engineering objectives. The results demonstrate promising performance, with AutoDev achieving remarkable scores of 91.5% and 87.8% for Pass@1 for code generation and test generation respectively. These outcomes underscore the effectiveness of AutoDev in au- tomating software engineering tasks while maintaining a secure and user-controlled development environment.

2 AutoDev Design
================

The design overview depicted in Fig. 2 provides insight into AutoDev's architecture. Once the initial configurations are complete, AutoDev organizes its capabilities into four groups: a Conversation Manager that tracks and manages the user and agents conversations; a Tools library where a variety of code and IDE related tools are available for the agents, an Agents Scheduler that schedules various agents, and an Evaluation Environment that enables execution operations. Below we explain each capability in details.

2.1 Rules, Actions, and Objective Configuration
-----------------------------------------------

The user initiates the process by configuring rules and ac- tions through yaml files. These files define the available commands (actions) that AI agents can perform. Users can leverage default settings or fine-grained permissions by en- abling/disabling specific commands, tailoring AutoDev to their specific needs. This configuration step allows for pre- cise control over the AI agents' capabilities. At this stage the user can define the number and behavior of the AI agents, assigning specific responsibilities, permissions, and available actions. For example, the user could define a "Developer" agent and a "Reviewer" agent, that collaboratively work to- wards an objective.

Following the rules and actions configuration, the user specifies the software engineering task or process to be ac- complished by AutoDev. For example, the user can ask to generate test cases and make sure they are syntactically correct, pass, and that do not contain bugs (this involves editing files, running test suite, executing syntax checks and bug-finding tools).

2.2 Conversation Manager
------------------------

The Conversation Manager, responsible for initializing the conversation history, plays a pivotal role in overseeing the high-level management of the ongoing conversation. It han- dles the task of deciding when to interrupt the process and en- sures seamless communication between the user, AI agents, and the system as a whole. It maintains a conversation ob- ject which includes the messages from the AI agents and the result of the actions from the Evaluation Environment.

2.2.1 Parser. The Parser interprets the responses gener- ated by agents, extracting commands and arguments in a predefined format. It ensures that the instructions are cor- rectly formatted, validating the number and accuracy of arguments (e.g., a file editing command requires the file path argument). In case of parsing failures, error messages are injected into the conversation, preventing further actions on the repository. Successfully parsed commands are fur- ther analyzed, by enforcing specific agent permissions and conducting additional semantic checks. It ensures that the suggested actions comply with the fine-grained permissions specified by the user. If the command passes scrutiny, the conversation manager invokes the corresponding action in the tools library.

2.2.2 Output Organizer. The Output Organizer module processes the output received from the Evaluation Environ- ment. It selects crucial information, such as status or errors, optionally summarizes relevant content, and adds a well- structured message to the conversation history. This ensures that the user has a clear and organized record of the Au- toDev's actions and outcomes.

<!-- PageHeader="AutoDev: Automated AI-Driven Development" --> <figure>

![](figures/0)

<!-- FigureContent="Rules & Actions Tools Library User Objective File Editing Conversation Manager Multi-Agents Eval Environment Retrieval :unselected: Conversation Agent Scheduler :unselected: State Build & Execution Parser Repository Testing :selected: +docker SECURITY Output Organizer Git" --> <figcaption>

Figure 2. Overview of the AutoDev Framework: The user initiates the process by defining the objective to be achieved. The Conversation Manager initializes the conversation and settings. The Agent Scheduler orchestrates AI agents to collaborate on the task and forwards their commands to the Conversation Manager. The Conversation Manager parses these commands and invokes the Tools Library, which offers various actions that can be performed on the repository. Agents' actions are executed within a secure Docker environment, and the output is returned to the Conversation Manager, which incorporates it into the ongoing conversation. This iterative process continues until the task is successfully completed.

</figcaption> </figure>

2.2.3 Conversation Conclusion. The Conversation Man- ager determines when to conclude the conversation. This may happen when an agent signals the completion of the task (stop command), the conversation reaches a user-defined maximum number of iterations/tokens, or issues are detected either in the process or within the Evaluation Environment. The comprehensive design of AutoDev ensures a systematic and controlled approach to AI-driven development.

2.3 Agent Scheduler
===================

The Agent Scheduler takes charge of orchestrating AI agents to achieve the user-defined objective. Agents, configured with specific personas and sets of available commands, oper- ate collaboratively to perform diverse tasks. The scheduler employs various collaboration algorithms, such as Round Robin, Token-Based, or Priority-Based, to determine the or- der and manner in which agents contribute to the conver- sation. Specifically, scheduling algorithms include but not limited to: (i) Round Robin collaboration, which invokes each agent sequentially, allowing each to execute a predeter- mined number of operations; (ii) token-based collaboration, enabling an agent to undertake multiple operations until it issues a token signifying completion of its assigned tasks; (iii) priority-based collaboration, initiating agents in accordance with their prioritized order. The agent scheduler invokes a specific agent passing the current conversation.

2.3.1 Agents. Agents, comprising Large Language Mod- els (LLMs) like OpenAI GPT-4 and Small Language Models (SLMs) optimized for code generation, communicate through textual natural language. These agents receive objectives and conversation histories from the Agent Scheduler, responding with actions specified by the Rules and Actions configura- tion. Each agent, with its unique configuration, contributes to the overall progress toward achieving the user's objective.

2.4 Tools Library
=================

The Tools Library within AutoDev provides a range of com- mands that empower Agents to perform diverse operations on the repository. These commands are designed to encapsu- late complex actions, tools, and utilities behind a straightfor- ward and intuitive command structure. For instance, intrica- cies related to build and test execution are abstracted away through simple commands like build and test <test\_file>.

· File Editing: This category encompasses commands for editing files, including code, configuration, and documentation. The utilities within this category, such as write, edit, insert, and delete, offer varying levels of granularity. Agents can perform actions ranging from writing entire files to modifying specific lines within a file. For example, the command write <filepath> <start\_line> -< end\_line> <content>, allows the agent to re-write a range of lines with new content.

<!-- PageHeader="Michele Tufano, et al." -->

· Retrieval: In this category, retrieval utilities range from basic CLI tools like grep, find, and ls to more so- phisticated embedding-based techniques. These tech- niques enable Agents to look up similar code snippets, enhancing their ability to retrieve relevant informa- tion from the codebase. For example, the command retrieve <content> allows the agent to perform embedding-based retrieval of similar snippets to the provided content.

. Build & Execution: Commands in this category allow Agents to compile, build, and execute the codebase effortlessly with simple and intuitive commands. The intricacies of low-level build commands are abstracted, streamlining the process within the Evaluation Envi- ronment infrastructure. Examples of commands in this category include: build, run <file>.

· Testing & Validation: These commands enable Agents to test the codebase by executing a single test case, a specific test file, or the entire test suite. Agents can perform these actions without relying on low-level commands specific to particular testing frameworks. This category also encompasses validation tools such as linters and bug-finding utilities. Examples of com- mands in this category include: syntax <file> which checks the syntax correctness, and test which runs the entire test suite.

· Git: Fine-grained permissions for git operations can be configured by the user. This includes operations like commits, push, and merges. For instance, Agents can be granted permission to perform only local com- mits or, if necessary, push changes to the origin repos- itory.

· Communication: Agents can invoke a set of com- mands aimed at facilitating communication with other agents and/or the user. Notably, the talk command enables sending natural language messages (not inter- preted as commands for repository actions), the ask command is used to request user feedback, and the stop command interrupts the process, indicating goal achievement or the inability of the agents to proceed further.

The Tools Library in AutoDev thus provides a versatile and accessible set of tools for AI Agents to interact with the codebase and communicate effectively within the collabora- tive development environment.

2.5 Evaluation Environment
==========================

Running within a Docker container, the Evaluation Envi- ronment allows secure execution of file editing, retrieval, build, execution, and testing commands. It abstracts away the complexity of low-level commands, providing a simplified interface for agents. The Evaluation Environment returns standard output/error to the Output Organizer module.

2.6 Putting Everything Together
===============================

The user initiates the conversation by specifying the ob- jective and associated settings. The conversation manager initializes a conversation object, consolidating messages from both AI agents and the Evaluation Environment. Sub- sequently, the conversation manager dispatches the conver- sation to the Agent Scheduler, responsible for coordinating the actions of AI agents. In their role as AI agents, Language Models (Large or Small LMs) suggest commands through textual interactions.

The Commands Interface encompasses a diverse set of functionalities, including File Editing, Retrieval, Build and Execution, Testing, and Git operations. These suggested com- mands are then parsed by the Conversation Manager, which subsequently directs them to the Evaluation Environment for execution on the codebase.

Execution of these commands occurs within the secure confines of the Evaluation Environment, encapsulated within a Docker container. Post-execution, the resulting actions seamlessly integrate into the conversation history, contribut- ing to subsequent iterations. This iterative process persists until the task is deemed complete by the Agents, user inter- vention occurs, or the maximum iteration limit is reached. AutoDev's design ensures a systematic and secure orchestra- tion of AI agents to achieve complex software engineering tasks in an autonomous and user-controlled manner.

3 Empirical Design
==================

In our empirical evaluation, we aim to assess AutoDev's ca- pabilities and effectiveness in software engineering tasks, ex- amining whether it can enhance an AI model's performance beyond simple inference. Additionally, we are interested in evaluating the cost of AutoDev in terms of the number of steps, inference calls, and tokens. We define three experi- mental research questions:

Research Questions
==================

1\. RQ1: How effective is AutoDev in code generation task?

2\. RQ2: How effective is AutoDev in test generation task?

3\. RQ3: How efficient is AutoDev in completing tasks?

RQ1: How effective is AutoDev in code generation task?
------------------------------------------------------

To address RQ1, we evaluate AutoDev's performance in a code generation task using the HumanEval problem-solving dataset in Python. This dataset comprises 164 handwritten programming problems, each containing a function signa- ture, docstring, body, and an average of 7.7 unit tests. In our assessment, AutoDev is provided with a partial file contain- ing the function signature and docstring, with the objective of implementing the method.

<!-- PageHeader="AutoDev: Automated AI-Driven Development" -->

Approach

Model Extra Training

Pass@1

Language Agent Tree Search

GPT-4

✓ :selected:

94.4

AutoDev

GPT-4

× :selected:

91.5

Reflexion

GPT-4

✓ :selected:

91.0

zero-shot (baseline)

GPT-4

× :selected:

67.0

Table 1. Code Generation results on HumanEval. AutoDev achieves top-3 performance on the leaderboard without extra training data, unlike LATS and Reflexion.

We gauge AutoDev's effectiveness using the Pass@k met- ric, where k represents the number of attempts made. A successfully solved problem is defined as one where Au- toDev generates the code of the method's body, satisfying all human-written tests. An attempt corresponds to an entire AutoDev conversation, which involves multiple inference calls and steps. This contrasts with other approaches, like directly invoking GPT-4, which typically involve a single inference call. Details regarding the multiple inference calls and steps are further explored in RQ3. For this evaluation, we set k = 1, thus computing Pass@1, considering only the success achieved in the first attempt.

RQ2: How effective is AutoDev in test generation task?
======================================================

For this research question, we modify the HumanEval dataset to evaluate AutoDev's capabilities in test generation. We con- sider the human-written solution and discard the provided human-written tests. AutoDev is instructed to generate test cases for the focal method and is evaluated based on test success, invocation of the focal method, and test coverage. We report Pass@1, considering tests successful if they pass and invoke the focal method. Additionally, we compare the coverage of AutoDev's tests with those written by humans.

RQ3: How efficient is AutoDev in completing tasks?
==================================================

In this research question, we investigate AutoDev's efficiency in completing SE tasks. We analyze the number of steps or inference calls needed, the distribution of commands used (e.g., write, test), and the total number of tokens used in the conversation.

AutoDev Settings
----------------

For this evaluation, AutoDev maintains consistent settings with one agent based on the GPT-4 model (gpt-4-1106-preview). Enabled actions include file editing, retrieval, and testing. The only communication command available is the stop command, indicating task completion. Other commands, such as ask, are disabled, requiring AutoDev to operate au- tonomously without human feedback or intervention beyond the initial goal-setting.

Approach

Model

Pass@1

Passing Coverage

Overall Coverage

Human

\-

100

99.4

99.4

AutoDev

GPT-4

87.8

99.3

88.8

zero-shot (baseline)

GPT-4

75

99.3

74

Table 2. Test Generation results on HumanEval. AutoDev enhances Pass@1 compared to baseline, generating tests achieving comparable coverage to human-written tests.

### 4 Empirical Results RQ1: How effective is AutoDev in a code generation task?

Table 1 displays the results for RQ1, comparing AutoDev against two alternative approaches and the zero-shot base- line. The table includes information about the model power- ing each approach, the need for additional training, and the Pass@1 metric.

We compared AutoDev with Language Agent Tree Search (LATS) and Reflexion, two leading approaches on the Hu- manEval leaderboard as of March 2024 \[1\]. The results for the zero-shot baseline (GPT-4) are taken from the OpenAI GPT-4 technical report \[11\], while those for LATS and Reflexion from the HuamnEval leaderboard \[1\].

Language Agent Tree Search (LATS) \[23\] is a versatile framework that utilizes Large Language Models (LLMs) for planning, acting, and reasoning. Inspired by Monte Carlo tree search, LATS employs LLMs as agents, value functions, and optimizers, repurposing their capabilities for improved decision-making.

Reflexion\[17\] introduces a unique framework to reinforce language agents via linguistic feedback without weight up- dates. Agents in Reflexion verbally reflect on task feedback signals, maintaining their reflective text in an episodic mem- ory buffer for enhanced decision-making.

Table 1 indicates that AutoDev achieves a Pass@1 rate of 91.5%, securing the second-best position on the HumanEval leaderboard. Notably, this result is obtained without addi- tional training data, distinguishing AutoDev from LATS, which achieves 94.4%. Furthermore, the AutoDev framework enhances GPT-4 performance from 67% to 91.5%, marking a 30% relative improvement.

These results underscore AutoDev's capability to signif- icantly enhance the performance of LLMs in completing software engineering tasks. However, it's worth noting that AutoDev's effectiveness may entail multiple inference calls and steps, as we delve into further detail in RQ3.

#### RQ2: How effective is AutoDev in test generation task?

Table 2 presents the outcomes for RQ2, comparing AutoDev with the zero-shot GPT-4 (baseline) and human-written tests in the test generation task. Since OpenAI has not evaluated

<!-- PageHeader="Michele Tufano, et al." -->

GPT-4 on the test generation task, we obtained the zero-shot GPT-4 results by invoking the inference with instruction prompting on the same GPT-4 model used for AutoDev.

AutoDev attains a Pass@1 score of 87.8% on the HumanEval dataset modified for the test generation task, exhibiting a 17% relative improvement over the baseline that utilizes the same GPT-4 model. The correct tests generated by AutoDev (included in Pass@1) achieve a robust 99.3% coverage, com- parable to the human-written tests' coverage of 99.4%. Addi- tionally, Table 2 reports the overall coverage across the entire dataset of test cases, considering incorrect or failing tests as lacking coverage. In this regard, AutoDev achieves an 88.8% coverage over the complete dataset of focal methods.

These results affirm AutoDev's prowess in addressing di- verse software engineering tasks.

##### RQ3: How efficient is AutoDev in completing tasks?

Figure 3 illustrates the cumulative number of commands used by AutoDev for both the Code Generation and Test Generation tasks, considering the average number of com- mands used for the evaluation of each HumanEval problem in RQ1 and RQ2.

For Code Generation, AutoDev executed an average of 5.5 commands, comprising 1.8 write operations, 1.7 test op- erations, 0.92 stop operations (indicating task completion), 0.25 incorrect commands, along with minimal retrieval (grep, find, cat), syntax check operations, and talk communication commands.

In the case of Test Generation, the average number of commands aligns with the Code Generation task. However, Test Generation involves more retrieval operations and an increased occurrence of incorrect operations, resulting in a total average of 6.5 commands for each run.

We classify a command as incorrect if it references an unavailable command or fails parsing (e.g., incorrect format- ting or parameter count). The most prevalent incorrect com- mands involve AI agents mixing natural language with code or commands. Such issues could potentially be addressed through more flexible parsing or improved prompting, as discussed further in the Discussion section.

While AutoDev incurs more inference calls compared to approaches that generate candidate code in a single call, it is essential to note that AutoDev also performs testing and validation operations, tasks typically executed by developers to validate generated candidates. Testing and syntax opera- tions invoked by AutoDev would be undertaken regardless by developers receiving code generated by AI, such as Copi- lot.

Furthermore, AutoDev often communicates task comple- tion through talk commands, providing insights and inter- pretability of the solution. Another notable communication command contributing to the overall count is the stop com- mand. This represents a relatively inexpensive inference call, generating only one token. Potential optimizations could

<figure>

![](figures/0)

<!-- FigureContent="7 Cumulative Number of Commands 6 5 1.38 grep 0.25 find 4 0.92 syntax 0.96 cat talk m 3 :unselected: incorrect 1.71 1.55 :unselected: stop test 2 write 1 1.85 1.91 0 Code Generation Test Generation Tasks" --> <figcaption>

Figure 3. Cumulative number of commands used by Au- toDev for an average task of Code and Test Generation

</figcaption> </figure>

involve batching such operations with other commands or inferences.

The average length of AutoDev conversations to solve each HumanEval problem in RQ1 and RQ2 is 1656 and 1863 tokens respectively. This encompasses the user's goal, mes- sages from the AI agent, and responses from the Evaluation Environment. In comparison, the zero-shot GPT-4 (baseline) uses 200 tokens (estimated) for code generation and 373 to- kens for test generation, on average, for each task. While AutoDev uses more tokens, a significant amount are spent for testing, validation, and explanation of its own generated code, going beyond what the baseline approach offers.

Finally, AutoDev incurs execution costs related to orches- trating AI Agents, managing conversations, and executing commands within a Docker environment. Notably, the Docker- based evaluation environment accounts for the major exe- cution cost, exhibiting higher overhead compared to direct CLI commands within the user's environment. However, we designed AutoDev with security as a top priority, ensuring the secure execution and validation of AI-generated code.

5 Discussion
============

5.1 AutoDev in Action
---------------------

Figures 4 (Part I) and 5 (Part II) provide a glimpse of Au- toDev in action during a test generation task, with some interactions summarized or omitted for brevity. Initiating from Figure 4, we set the goal for AutoDev to generate pytest test cases following a specific format. The AutoDev agent initiates the write-new command, providing the filepath and content of the test file. AutoDev executes the operation, confirming successful content writing to the agent. Subse- quently, the AutoDev agent triggers the test operation, and AutoDev runs the test within its secure Docker environment, presenting the test execution report JSON (summarized in the figure for brevity).

<!-- PageHeader="AutoDev: Automated AI-Driven Development" -->

User - Objective
================

Write a Pytest test with 4-5 assertions that covers the following function, located at human\_answer. py:

def is\_bored(S):

You'll be given a string of words, and your task is to count the number of boredoms. A boredom is a sentence that starts with the word "I". Sentences are delimited by '. ', '?' or '!'.

For example: >>> is\_bored("Hello world") 0

\>>> is\_bored("The sky is blue. The sun is shining. I love this weather") 1

import re

sentences = re. split (r'\[. ?! \]\\s\*', S) return sum(sentence\[0:2\] == 'I ' for sentence in sentences) Write the test to a new file /HumanEval\_91/test\_HumanEval\_91.py. The test method should be written in the following format:

from .human\_answer import \* def test\_function\_name () : # assertions for the given function here

Run your generated test and confirm it passes.

AutoDev - Agent

write-new /HumanEval\_91/test\_HumanEval\_91.py from .human\_answer import is\_bored import pytest

def test\_is\_bored(): assert is\_bored('Hello world') == 0 assert is\_bored('I am bored. This is boring! ') == 2 assert is\_bored('The sky is blue. The sun is shining. I love > this weather . ') = 1 assert is\_bored('I think, therefore I am. I am bored?') == 2 assert is\_bored('') == 0

AutoDev - Environment
=====================

Content successfully written to /HumanEval\_91/test\_HumanEval\_91.py AutoDev - Agent

test

AutoDev - Environment
=====================

'summary': {'failed': 1, 'total': 1, 'collected': 1}

'message' : "AssertionError: assert 1 == 2\\n + where 1 = > is\_bored('I am bored. This is boring!')

AutoDev - Agent
===============

The test case that failed expected 'is\_bored('I am bored. This is > boring!' )" to return "2", but according to the function

\> implementation, it should return "1" because the function

\> counts sentences that start with the word "I" followed by a space. The sentence "This is boring!" does not start with "I", so it should not be counted.

The test case is incorrect and needs to be fixed. Let's correct > the test case to match the expected behavior of the function.

AutoDev - Agent

write /HumanEval\_91/test\_HumanEval\_91.py 5-5

" assert is\_bored('I am bored. This is boring!') == 1"

AutoDev - Environment

File updates successfully.

AutoDev - Agent

test

AutoDev - Environment

'summary': {'passed': 1, 'total': 1, 'collected': 1}

AutoDev - Agent

The test suite has passed successfully. All tests for the > \`is\_bored" function are now correct and the function behaves > as expected. The goal has been reached.

Figure 5. AutoDev in Test Generation scenario (part II)

The AutoDev agent identifies an error in the pytest output, recognizing that a fix is required to align the test with the expected behavior of the function.

Continuing in Figure 5, the AutoDev agent issues the write command, specifying the filepath and line number range (5-5) for rewriting the incorrect assert statement. Fol- lowing this, the AutoDev agent proceeds with the test exe- cution, which now succeeds. Recognizing the completion of the goal, the AutoDev agent concludes the conversation.

This example highlights AutoDev's capability to self-evaluate its generated code and address bugs within its own output. Additionally, it demonstrates how AutoDev facilitates user insight into agent actions, allowing agents to communicate during the task.

5.2 Multi-Agent Collaboration
-----------------------------

AutoDev supports multi-agent collaboration on tasks, or- chestrated by the Agent Scheduler. In our evaluation, given the relative simplicity of the HumanEval dataset, we limited our setup to a single GPT-4 agent for the tasks.

However, preliminary results indicate the positive impact of multi-agent collaboration on more complex tasks. Experi- menting with an AI Developer and AI Reviewer, each with distinct responsibilities and available actions, collaborating on fixing a complex bug revealed interesting interactions. The AI Reviewer could pre-emptively identify AI Developer mistakes before code validation actions were executed and provide relevant suggestions.

Our future plans involve expanding evaluations to incor- porate more complex scenarios where multi-agent collabo- ration can significantly enhance AutoDev's performance.

<!-- PageFooter="Figure 4. AutoDev in Test Generation scenario (part I)" --> <!-- PageHeader="Michele Tufano, et al." -->

5.3 Human in the Loop
---------------------

AutoDev allows AI agents to communicate progress on tasks or request human feedback using the talk and ask com- mands, respectively. Anecdotally, these commands have proven helpful for developers using AutoDev to understand the agent's intentions and gain insights into the agent's plan. The addition of the ask command was in direct response to a developer's request during our pilot study, where they wanted the ability to provide feedback when the agents ap- peared uncertain about the next actions.

Our future plans involve deeper integration of humans within the AutoDev loop, allowing users to interrupt agents and provide prompt feedback.

5.4 AutoDev Integrations
------------------------

Our pilot study involved developers using AutoDev as a CLI command, with the conversation available for observation within the VSCode IDE.

Moving forward, our goal is to integrate AutoDev into IDEs, creating a chatbot experience, and incorporate it into CI/CD pipelines and PR review platforms. We envision de- velopers assigning tasks and issues to AutoDev, reviewing results within a PR system, and further streamlining the software development workflow.

6 Related Work
==============

Our work builds upon an extensive body of literature that applies AI to various Software Engineering tasks. In this section, we explore recent developments and contextualize AutoDev within this rich research landscape.

6.1 AI in Software Engineering
------------------------------

The integration of AI, particularly Large Language Models (LLMs), into software engineering has witnessed substantial progress. Models like GPT-3 \[7\], InstructGPT \[14\], and GPT- 4 \[13\] have leveraged the Transformer architecture \[20\] to comprehend and generate not only natural language, but also source code. The massive parameter sizes of LLMs, such as those in Gropher \[15\] and Megatron-turing NLG \[18\], and GPT-4 \[13\] allowed these AI models to achieve impressive performance across diverse tasks.

As software development practices continue to evolve, the integration of cutting-edge technologies becomes para- mount for enhancing developer productivity \[5\]. Among the notable advancements, the utilization of LLMs within Integrated Development Environments (IDEs) has garnered significant attention \[4, 10\]. LLMs, including prominent mod- els such as OpenAI's GPT-3.5 \[12\] and GPT-4 \[13\], as well as robust open-source models like Code Llama \[16\], exhibit the potential to act as intelligent programming assistants.

In this paper, we introduce AutoDev, a comprehensive framework for autonomous software engineering tasks within

a secure development environment. AutoDev extends be- yond existing works by providing a versatile tools library, empowering AI agents to autonomously perform intricate tasks, such as code editing, testing, and integration. AutoDev is also LLM-agnostic, with an infrastructure that allows a diverse set of AI models, with different parameter size and architectures, to collaborate on a given task.

6.2 Evaluation of LLMs in Software Engineering
----------------------------------------------

Evaluating LLMs for software engineering tasks poses unique challenges. Traditional language-based metrics, such as BLEU, have been the focus of previous research, with evaluations conducted on static datasets like GLUE \[21\] and BIGBench \[19\]. However, these metrics often fall short in capturing essential programming aspects like syntax correctness and execution-based metrics such as build and testing.

CodeXGLUE \[9\] has addressed these limitations by pro- viding a comprehensive evaluation platform for LLMs in software engineering. It offers a diverse benchmark dataset along with baseline models like CodeBERT and CodeGPT.

HumanEval \[5\] contributes to the field by focusing on the functional correctness of LLMs, introducing a benchmark dataset of hand-written programming problems in Python.

In recent developments, the Copilot Evaluation Harness \[3\] builds upon prior works in the literature, aiming to en- hance their contributions. Similar to HumanEval, the Copilot Evaluation Harness incorporates considerations of code exe- cution, but extending the spectrum of software engineering tasks (code, test, and documentation generation, workspace understanding and query resolution) as well as increasing the metrics used for evaluation. This evaluation harness also encompasses large and real-world codebases.

While our current evaluation relies on HumanEval for assessing the effectiveness of AutoDev in two coding tasks, our future work aims to extend this assessment to more challenging and real-world datasets, such as those offered by the Copilot Evaluation Harness.

6.3 AI in Software Engineering Interactions
-------------------------------------------

While prior works have explored the intersection of AI and software engineering, few have delved into AI-guided pro- gramming within IDE interactions. AutoDev, as introduced in this paper, draws inspiration from existing works in the literature while enhancing their contributions. Notable ex- amples include Auto-GPT\[8\], LATS (Language Agent Tree Search)\[23\], and Reflexion\[17\], each presenting unique ap- proaches to AI-driven tasks.

Auto-GPT\[8\] operates by pairing GPT-3.5 and GPT-4 with a companion bot, enabling users to instruct these language models on specific goals. The companion bot utilizes GPT- 3.5 and GPT-4, along with various programs, to execute the necessary steps for goal achievement.

LATS \[23\], on the other hand, is a general framework that synergizes the capabilities of LLMs in planning, acting, and SERVICES
DOCUMENTS
AutoDev: Automated AI-Driven Development.md
PREVIEW AS 
EXPORT AS 
SAVE TO 
IMPORT FROM 
DOCUMENT NAME
AutoDev: Automated AI-Driven Development.md
READING TIME: 9 MIN READ WORDS: 1629CHARACTERS: 10520
MARKDOWNToggle Zen ModePREVIEWToggle Mode

<!-- PageHeader=“AutoDev: Automated AI-Driven Development” -->

reasoning. Inspired by Monte Carlo tree search, commonly used in model-based reinforcement learning, LATS employs LLMs as agents, value functions, and optimizers, enhancing decision-making. It introduces an environment for external feedback, offering a deliberate and adaptive problem-solving mechanism.

Reflexion[17] introduces a novel framework to reinforce language agents through linguistic feedback. Reflexion agents verbally reflect on task feedback signals, maintaining their reflective text in an episodic memory buffer for improved decision-making. This flexible framework incorporates vari- ous types and sources of feedback signals and exhibits sig- nificant improvements over baseline agents in diverse tasks, including sequential decision-making, coding, and language reasoning.

AutoDev specializes these ideas for the Software Engi- neering realm, offering a flexible framework that allows AI agents to complete complex SE tasks in full autonomy. Our work aims to bridge the gap between traditional software engineering practices and AI-driven automation, facilitating collaborative efforts between developers and AI agents. By introducing a versatile tools library, AutoDev empowers AI agents to autonomously perform intricate tasks, providing a promising advancement in the landscape of AI-assisted software development.

7 Conclusion
In this paper, we introduced AutoDev, a framework enabling AI Agents to autonomously interact with repositories, per- form actions, and tackle complex software engineering tasks. We’ve shifted the responsibility of extracting relevant con- text for software engineering tasks and validating AI-generated code from users (mainly developers) to the AI agents them- selves. Agents are now empowered to retrieve context through Retrieval actions and validate their code generation through Build, Execution, Testing, and Validation actions.

The developer’s role within the AutoDev framework trans- forms from manual actions and validation of AI suggestions to a supervisor overseeing multi-agent collaboration on tasks, with the option to provide feedback. Developers can monitor AutoDev’s progress toward goals by observing the ongoing conversation used for communication among agents and the repository.

Our evaluation on the HumanEval dataset for code and test generation showcased impressive results, achieving a Pass@1 score of 91.5 for code generation-a second-best result on the leaderboard at the time of writing, and the best among approaches requiring no extra training data. AutoDev also excelled in test generation with a Pass@1 score of 87.8%, achieving a 99.3% coverage from passing tests.

Looking ahead, our goal for future work is to integrate AutoDev into IDEs as a chatbot experience and incorporate it into CI/CD pipelines and PR review platforms.

References
[1] Code generation on humaneval - state-of-the-art. https:// paperswithcode.com/sota/code-generation-on-humaneval, 2024. Ac- cessed: 2024-02-27.

[2] Github copilot: Your ai pair programmer. https://github.com/features/ copilot, 2024.

[3] AGARWAL, A., CHAN, A., CHANDEL, S., JANG, J., MILLER, S., MOGHAD- DAM, R. Z., MOHYLEVSKYY, Y., SUNDARESAN, N., AND TUFANO, M. Copi- lot evaluation harness: Evaluating llm-guided software programming, 2024.

[4] CHEN, B., MUSTAKIN, N., HOANG, A., FUAD, S., AND WONG, D. Vscuda: Llm based cuda extension for visual studio code. In Proceedings of the SC '23 Workshops of The International Conference on High Performance Computing, Network, Storage, and Analysis (New York, NY, USA, 2023), SC-W '23, Association for Computing Machinery, p. 11-17.

[5] CHEN, M., TWOREK, J., JUN, H., YUAN, Q., DE OLIVEIRA PINTO, H. P., ET AL. Evaluating large language models trained on code.

[6] DING, Y., WANG, Z., AHMAD, W., DING, H., TAN, M., JAIN, N., RA- MANATHAN, M. K., NALLAPATI, R., BHATIA, P., ROTH, D., ET AL. Cross- codeeval: A diverse and multilingual benchmark for cross-file code completion. Advances in Neural Information Processing Systems 36 (2024).

[7] FLORIDI, L., AND CHIRIATTI, M. Gpt-3: Its nature, scope, limits, and consequences. Minds and Machines 30 (2020), 681-694.

[8] GRAVITAS, S. Autogpt. https://github.com/Significant-Gravitas/ AutoGPT, 2024. GitHub repository.

[9] LU, S., GUO, D., REN, S., HUANG, J., SVYATKOVSKIY, A., BLANCO, A., CLEMENT, C., DRAIN, D., JIANG, D., TANG, D., LI, G., ZHOU, L., SHOU, L., ZHOU, L., TUFANO, M., GONG, M., ZHOU, M., DUAN, N., SUNDARESAN, N., DENG, S. K., FU, S., AND LIU, S. Codexglue: A machine learning benchmark dataset for code understanding and generation, 2021.

[10] NAM, D., MACVEAN, A., HELLENDOORN, V., VASILESCU, B., AND MYERS, B. In-ide generation-based information support with a large language model, 2023.

[11] OPENAI, :, ACHIAM, J., ADLER, S., AGARWAL, S., AHMAD, L., AKKAYA, I., ALEMAN, F. L., ALMEIDA, D., ALTENSCHMIDT, J., ALTMAN, S., ANADKAT, S., AVILA, R., BABUSCHKIN, I., BALAJI, S., BALCOM, V., BALTESCU, P., BAO, H., BAVARIAN, M., BELGUM, J., BELLO, I., BERDINE, J., BERNADETT- SHAPIRO, G., BERNER, C., BOGDONOFF, L., BOIKO, O., BOYD, M., BRAK- MAN, A .- L., BROCKMAN, G., BROOKS, T., BRUNDAGE, M., BUTTON, K., CAI, T., CAMPBELL, R., CANN, A., CAREY, B., CARLSON, C., CARMICHAEL, R., CHAN, B., CHANG, C., CHANTZIS, F., CHEN, D., CHEN, S., CHEN, R., CHEN, J., CHEN, M., CHESS, B., CHO, C., CHU, C., CHUNG, H. W., CUM- MINGS, D., CURRIER, J., DAI, Y., DECAREAUX, C., DEGRY, T., DEUTSCH, N., DEVILLE, D., DHAR, A., DOHAN, D., DOWLING, S., DUNNING, S., ECOFFET, A., ELETI, A., ELOUNDOU, T., FARHI, D., FEDUS, L., FELIX, N., FISHMAN, S. P., FORTE, J., FULFORD, I., GAO, L., GEORGES, E., GIBSON, C., GOEL, V., GOGINENI, T., GOH, G., GONTIJO-LOPES, R., GORDON, J., GRAFSTEIN, M., GRAY, S., GREENE, R., GROSS, J., GU, S. S., Guo, Y., HALLACY, C., HAN, J., HARRIS, J., HE, Y., HEATON, M., HEIDECKE, J., HESSE, C., HICKEY, A., HICKEY, W., HOESCHELE, P., HOUGHTON, B., Hsu, K., HU, S., HU, X., HUIZINGA, J., JAIN, S., JAIN, S., JANG, J., JIANG, A., JIANG, R., JIN, H., JIN, D., JOMOTO, S., JONN, B., JUN, H., KAFTAN, T., ŁUKASZ KAISER, KAMALI, A., KANITSCHEIDER, I., KESKAR, N. S., KHAN, T., KILPATRICK, L., KIM, J. W., KIM, C., KIM, Y., KIRCHNER, J. H., KIROS, J., KNIGHT, M., KOKOTAJLO, D., ŁUKASZ KONDRACIUK, KONDRICH, A., KONSTANTINIDIS, A., KOSIC, K., KRUEGER, G., KUO, V., LAMPE, M., LAN, I., LEE, T., LEIKE, J., LEUNG, J., LEVY, D., LI, C. M., LIM, R., LIN, M., LIN, S., LITWIN, M., LOPEZ, T., LOWE, R., LUE, P., MAKANJU, A., MAL- FACINI, K., MANNING, S., MARKOV, T., MARKOVSKI, Y., MARTIN, B., MAYER, K., MAYNE, A., MCGREW, B., MCKINNEY, S. M., McLEAVEY, C., McMILLAN, P., McNEIL, J., MEDINA, D., MEHTA, A., MENICK, J., METZ, L., MISHCHENKO, A., MISHKIN, P., MONACO, V., MORIKAWA, E., MOSSING, D., MU, T., MURATI, M., MURK, O., MÉLY, D., NAIR, A., :selected: :selected: :selected:

<!-- PageHeader=“Michele Tufano, et al.” -->

NAKANO, R., NAYAK, R., NEELAKANTAN, A., NGO, R., NOH, H., OUYANG, L., O’KEEFE, C., PACHOCKI, J., PAINO, A., PALERMO, J., PANTULIANO, A., PARASCANDOLO, G., PARISH, J., PARPARITA, E., PASSOS, A., PAVLOV, M., PENG, A., PERELMAN, A., DE AVILA BELBUTE PERES, F., PETROV, M., DE OLIVEIRA PINTO, H. P., MICHAEL, POKORNY, POKRASS, M., PONG, V. H., POWELL, T., POWER, A., POWER, B., PROEHL, E., PURI, R., RADFORD, A., RAE, J., RAMESH, A., RAYMOND, C., REAL, F., RIMBACH, K., Ross, C., ROTSTED, B., ROUSSEZ, H., RYDER, N., SALTARELLI, M., SANDERS, T., SANTURKAR, S., SASTRY, G., SCHMIDT, H., SCHNURR, D., SCHULMAN, J., SELSAM, D., SHEPPARD, K., SHERBAKOV, T., SHIEH, J., SHOKER, S., SHYAM, P., SIDOR, S., SIGLER, E., SIMENS, M., SITKIN, J., SLAMA, K., SOHL, I., SOKOLOWSKY, B., SONG, Y., STAUDACHER, N., SUCH, F. P., SUMMERS, N., SUTSKEVER, I., TANG, J., TEZAK, N., THOMPSON, M. B., TILLET, P., TOOTOONCHIAN, A., TSENG, E., TUGGLE, P., TURLEY, N., TWOREK, J., URIBE, J. F. C., VALLONE, A., VIJAYVERGIYA, A., Voss, C., WAINWRIGHT, C., WANG, J. J., WANG, A., WANG, B., WARD, J., WEI, J., WEINMANN, C., WELIHINDA, A., WELINDER, P., WENG, J., WENG, L., WIETHOFF, M., WILLNER, D., WINTER, C., WOLRICH, S., WONG, H., WORKMAN, L., WU, S., WU, J., WU, M., XIAO, K., XU, T., Yoo, S., YU, K., YUAN, Q., ZAREMBA, W., ZELLERS, R., ZHANG, C., ZHANG, M., ZHAO, S., ZHENG, T., ZHUANG, J., ZHUK, W., AND ZOPH, B. Gpt-4 technical report, 2024.

[12] OPENAI. Gpt 3.5 models, 2023.

[13] OPENAI. Gpt-4 technical report, 2023.

[14] OUYANG, L., WU, J., JIANG, X., ALMEIDA, D., WAINWRIGHT, C., MISHKIN, P., ZHANG, C., AGARWAL, S., SLAMA, K., RAY, A., ET AL. Training lan- guage models to follow instructions with human feedback. Advances in Neural Information Processing Systems 35 (2022), 27730-27744.

[15] RAE, J. W., BORGEAUD, S., CAI, T., MILLICAN, K., AND OTHERS. Scaling language models: Methods, analysis & insights from training gopher, 2022.

[16] ROZIERE, B., GEHRING, J., GLOECKLE, F., SOOTLA, S., GAT, I., TAN, X. E., ADI, Y., LIU, J., REMEZ, T., RAPIN, J., ET AL. Code llama: Open foundation models for code. arXiv preprint arXiv:2308.12950 (2023).

[17] SHINN, N., CASSANO, F., BERMAN, E., GOPINATH, A., NARASIMHAN, K., AND YAO, S. Reflexion: Language agents with verbal reinforcement learning, 2023.

[18] SMITH, S., PATWARY, M., NORICK, B., LEGRESLEY, P., RAJBHANDARI, S., CASPER, J., LIU, Z., PRABHUMOYE, S., ZERVEAS, G., KORTHIKANTI, V., ZHANG, E., CHILD, R., AMINABADI, R. Y., BERNAUER, J., SONG, X., SHOEYBI, M., HE, Y., HOUSTON, M., TIWARY, S., AND CATANZARO, B. Using deepspeed and megatron to train megatron-turing nlg 530b, a large-scale generative language model, 2022.

[19] SRIVASTAVA, A., RASTOGI, A., RAO, A., SHOEB, A. A. M., ET AL. Beyond the imitation game: Quantifying and extrapolating the capabilities of language models, 2023.

[20] VASWANI, A., SHAZEER, N., PARMAR, N., USZKOREIT, J., JONES, L., GOMEZ, A. N., KAISER, Ł., AND POLOSUKHIN, I. Attention is all you need. Advances in neural information processing systems 30 (2017).

[21] WANG, A., SINGH, A., MICHAEL, J., HILL, F., LEVY, O., AND BOWMAN, S. R. Glue: A multi-task benchmark and analysis platform for natural language understanding, 2019.

[22] WU, Q., BANSAL, G., ZHANG, J., WU, Y., LI, B., ZHU, E., JIANG, L., ZHANG, X., ZHANG, S., LIU, J., AWADALLAH, A. H., WHITE, R. W., BURGER, D., AND WANG, C. Autogen: Enabling next-gen Ilm applications via multi- agent conversation, 2023.

[23] ZHOU, A., YAN, K., SHLAPENTOKH-ROTHMAN, M., WANG, H., AND WANG, Y .- X. Language agent tree search unifies reasoning acting and planning in language models, 2023.