AI Portfolio Chatbot (“Ask Me Anything”)

An interactive AI assistant that answers questions about my background while demonstrating how I design controlled, reliable agent behavior.

⸻

What this is

This is a deployed AI chatbot that lives on my portfolio website and allows visitors to ask questions about my experience, skills, and background in a natural, conversational way.

Instead of navigating through multiple pages, users can simply ask what they want to know and get a direct answer. The bot also allows visitors to request my resume, which is sent via email and logged so I can follow up.

⸻

Why I built this

Traditional portfolio sites require people to search for information and piece things together over time. I wanted to create something more direct and useful, where someone could quickly understand my experience without friction.

This project started as a course exercise, but I expanded it into a real system. It became both a learning tool and a practical way to demonstrate how I think about building AI-driven workflows that are actually usable.

⸻

How it works

The chatbot is grounded in a curated set of context files that include my resume, background information, skills, and portfolio content. This allows it to respond in first person, in my voice, while staying anchored to information I control.

On top of that, I introduced a tool-based system to handle specific tasks more reliably. Instead of relying on the model to reason through everything, the chatbot can call structured functions when needed.

Key tools include:

* get_employment_history
    Returns structured employment data filtered by duration and role type. This ensures questions about my work history are answered consistently and accurately.
* send_resume
    Emails my resume to the user and notifies me when a request is made.
* record_unknown_question
    Logs unanswered or unclear questions so I can improve the system over time.

All conversations are also logged for review, which helps me monitor quality and identify areas where the system can be improved.

⸻

Key design decisions

One of the most important shifts in this project was realizing that the model could not reliably reason over certain types of data, even when that data was clearly provided.

For example, when asked how many companies I had worked at for a certain duration, the model would give inconsistent answers due to overlapping roles. No amount of prompt tuning fully fixed this.

Instead of continuing to rely on the model, I moved that logic into code. The chatbot now uses a structured tool to handle employment data, removing the need for the model to interpret or calculate anything.

I also avoided having the model rewrite or summarize content unnecessarily. The goal was to preserve accuracy and keep responses grounded in real information, not generated approximations.

Finally, I added post-processing layers to control formatting and remove unwanted behaviors, such as inconsistent lists or repetitive closing statements. This ensured the output remained clean and predictable.

⸻

Challenges

The biggest challenge was learning to move from “letting the AI handle it” to designing a system that controls how the AI behaves.

Early versions of the chatbot worked, but not in a way that was easy to trust or improve. The model would produce inconsistent outputs, ignore formatting instructions, and occasionally misinterpret structured data.

By introducing tools, validation layers, and post-processing, I was able to make the system more reliable and easier to reason about. This shift in approach was one of the most valuable parts of the project.

⸻

The result

The final system allows users to interact with my portfolio in a more direct and flexible way while also demonstrating how I approach building AI systems.

Rather than relying entirely on the model, the chatbot uses a combination of structured data, controlled tool usage, and post-processing to produce more consistent and reliable results.

⸻

What’s next

Future improvements include expanding the knowledge base, automating resume delivery fully, and continuing to refine how responses are generated and controlled.

More broadly, this project serves as a foundation for building more advanced agent-based systems that combine conversational interfaces with structured workflows and real-world actions.

⸻

Tech stack

* Python
* OpenAI (GPT-4o)
* Gradio
* Hugging Face Spaces
* Gmail SMTP
* Google Sheets API

⸻

What this project demonstrates

* grounding AI systems in controlled data
* using tools to replace unreliable model reasoning
* designing workflows around AI instead of relying on prompts alone
* improving reliability through structure and post-processing