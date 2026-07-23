
## Curious Learner

I'm a curious learner. Whenever I have some flexibility before a deadline, instead of just exploiting the tools at my disposal to reach a solution as quickly as possible, I try to explore and learn about unfamiliar terminology and concepts by peeling away at least a few layers of abstraction until reaching a point of familiarity. Then I move on towards the solution. Even when that deeper understanding does not immediately change the current solution, it often helps me approach future problems with more insight.

## Side hobby

Drawn to the mechanics of language—from grammatical elements such as punctuation and parenthetical expressions to precise terminology, vocabulary, and idiomatic expressions—with the goal of improving clarity, phrasing, and readability.

## Describing AI Experience

In my most recent internship, I took full ownership of  developing AI products with features that involved RAG retrieval of NIST control embeddings, fallback mechanism across different models and guardrails for efficiency and memory management.

In another internship, I built an AI augmented submission review feature that was integrated into a larger system designed for very low idle cost of operation.

I have also built worked on multiple projects in the AI domain ranging from a game that had a natural language command parsing feature, a system that allowed implementing lexical constraints in seq2seq generation and a research analysis of multiple computer vision models to see which one performs best on a certain watermarking technique

## AI Specific Technologies

- Core Open Source libraries: Pytorch, Tensorflow, Huggingface Transformer.
- Core Open Source Protocol: MCP (For tools and function calling)
- Chat and Embedding models: OpenAI, Claude, Ollama.
- Open Source AI Frameworks: LangChain, Crew AI
- Vector databases: Qdrant, Pinecone.

---

- Driven by impact, energized by challenge,
- Able to learn and adapt to solve evolving problems.
- Wanna make impact in high stakes industries like finance, manufacturing, defense, healthcare and public sector and make AI systems for them that run on their own terms and give them ownership and end-to-end control of the product instead of outsourcing Agentic workload to vendors like OpenAI or Claude.
- Try to Rapidly prototype and deliver POCs and iterate on solutions using an experimentation-driven engineering approach.  

## Why do you want to join a research fellowship(Anthropic)

In the past two years, my favorite moments have been spent learning while working on a research project of my choice and that choice is usually centered around LLMs or Computer vision models. In my research project "Nothing Lost in Translation", I learned about the core internals of the decoding process by building a grid beam search decoder that could implement lexical constraints in Seq2Seq generation. It ended up pushing the works of the "Hokamp & Liu(2017)" paper forward and showed implementation of positive constraints in the specific task of English to Russian Machine Translation but also showed negative constraint implementation that was not covered in the original paper's implementation and would be helpful for downstream tasks like content sanitation and age appropriateness in translation, hence guaranteeing safer translation using AI. Even my baseline decoder outperformed Huggingface's .generate() function by 1.45 BLEU score. In another research project - "When Pixels Talk Back", I discovered that Computer Vision models are least negatively affected by a specific blind fragile watermarking technique and that makes it especially helpful for embedding EMR data into CT scans. In another one I built a natural language command parser for a custom Text adventure game I built. I have also worked on industry applications of LLM automation in my internships dealing with building RAG pipelines, fallback mechanisms, Agent orchestration, guardrails and debugging memory issues by inspecting the model weights in OS page cache.

But I believe there was one recurring limitation  of "resources and compute" across my research projects and internship work. I believe I could have done way more in all the projects mentioned above like showing large scale content sanitation for translation, comparing results of the watermarking with larger contemporary industry standard computer vision models or building a command parser for the game that was able to generate alternate subplots instead of following precoded ones. Since Anthropic's Fellows program largely takes away those limitations and adds guidance of highly skilled mentorship it is the best option available to me to pursue my personal interest in understanding the core mechanics of model internals and make an impact by improving performance and advancing safer use of such technologies.

**Question - Tell us briefly about one or more research areas you're excited about right now, and why.**

Keeping in mind the context I gave about my prior research and professional experience in the previous answer, I would want to work on things like figuring out how far decode-time methods can scale, whether constraint mechanisms can be made cheap enough for production inference, and whether they can incorporate richer policies than just lexical constraints. This excites me because I think decode time interventions offer hard guarantees on model outputs that serves as complementary safety artifact to fine-tuning's probabilistic measures. This is why my top choices are "ML Systems and performance" and a close second being "AI safety and alignment". Another area could be working on developing/improving the infrastructure underneath related empirical research itself.
