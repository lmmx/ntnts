Here are all the questions from the two interviews, numbered and paired with your verbatim answers:

## Interview A

**A1.** Is the relationship between `octopolars` and `ntnts` meant to be:
- A direct dependency where `ntnts` builds upon `octopolars`?
- A spiritual successor with similar design principles but different focus?
- An entirely separate tool that just happens to share some technical approaches?

**Your answer:**
"no so octopolars was just a nicely done project, it had good packaging, good CLI pattern, and is an (optional) dependency of the new project so it makes sense to bootstrap the new one from the skeleton of the old one, both in the readme and the code. you will be able to use extras to put octopolars in it, plus the polars vs. polars-lts-cpu extras will be maintained from the octopolars (an artifact of different support for certain CPU instructions on different hardware, that part can just be brought straight in from the old readme). it is a spiritual successor in the sense that it forms part of a stack on top of the octopolars project (plus polars-ls, another of my python packages with a similar polars-oriented design). it should be noted though that it is entirely separate in the sense of being unconnected software, it operates at a very different purpose (to do with understanding software whereas octopolars is just a logistical aspect of getting the repo files, and for that we can use polars-ls to get them from a local directory instead)."

**A2.** What is the primary user problem that `ntnts` solves that existing tools don't address adequately?

**Your answer:**
"the primary user problem ntnts solves is of a desire for code-use: actions, with too many distractions, LLMs are the only game in town, but they can't stop getting pushed off their context window focus on a naive "code stuffing" wave. the recent mutagrep paper outlined a highly effective way to upgrade this model of dumping entire repos into LLMs before trying to generate code (edits with new functionality or bugfixes and so on). there have been increasing levels of repo maps (like aider or griffe which use object inventories), llms.txt (which aimed to just tell the LLM manually what to do with the repo, which obviously won't scale, or at least it isn't quite a self-contained causal framework for *how* to tell the LLM what to do with it). these approaches are in their own ways insufficient, but point towards a goal of automating the act of understanding, and mutagrep takes the approach of *grounding* it in symbols (it calls it symbol mining but we are really just talking about AST mining, pulling out the symbols, using the corresponding source to generate action-oriented *purposes* of the symbol in what feels like an intuitively semantic way similar to how humans think about the code as having specific purposes, or being *used* in action-oriented ways to achieve particular subgoals, and this naturally lends itself to decomposing subtasks). the mutagrep paper is great but it is a full implementation and has all the degrees of freedom baked in and I want to make a new system with in some ways more simplicity, more sophisticated packaging, and aiming to reproduce the mutagrep work with open source tools (like fastembed rather than the openai API for embeddings). the idea behind it is to build on existing polars (dataframe) powertools of octopolars, polars-ls, and polars-fastembed. that way the project becomes relatively simple, just orchestrating the combination of source code and prompts. it could all be so simple, if we just get out of the way of the software, and dont limit the software to how long the maintainers can maintain the context of its functionality in their head until their working memory runs out of battery. in short, the tool will just output a dataframe, simple. the dataframe will be clearly structured from per-repo dataframe unit -> rows in the df correspond to the files in the repo -> we make some multiple [array dtype in the polars schema] of intents, typically 5, so a 5D array of "intents" (this is a specific way of naming the string descriptions with an action-oriented natural language exposition of the purpose of the code that naturally lends itself to subtask decomposition when paired with a pyflakes-style symbol mining process, i.e. AST node visitor walking, and then those intents get condensed into a singular vector [representing that 5D/however-many dimensions array of intents, not one vector per intent dimension]. this dataframe will then be able to be enriched in various ways to explore, represent, rerank, or retrieve from. this will be extremely practical for code-use tasks. this was the general idea of the paper."

**A3.** The spec mentions "execution-free, repository-grounded" analysis - could you clarify what this means in practical terms for users?

**Your answer:**
"execution-free means that the code does not need to execute (unlike other forms of inventory mining such as mkdocs Griffe, which is used to ensure proper type resolution but can limit usage on software with heavyweight dependencies or simply prevent scaling across many codebases as execution will necessarily slow down any approach compared to a purely static analysis). repository-grounded refers to grounding the downstream task in the symbols [and an extracted semantic interpretation, in our case a vector embedding of the 'intents', an LLM-generated text representation (typically 5 redundant variants)], for downstream tasks like reordering/reranking, retrieval, clustering, and so on. this is practically useful for instance when you want to ask questions (natural language text queries) or find ('retrieve') things in a codebase that is either so large as to be difficult to intuitively navigate, or simply unfamiliar."

**A4.** Which components of the `octopolars` approach should be preserved in `ntnts`? (e.g., caching strategy, DSL expressions, Polars integration)

**Your answer:**
"the components of octopolars preserved in ntnts is to offload anything to do with the logistical aspect of going to a github repo and pulling/walking its files and extracting their contents. caching of repos is thus offloaded too, the DSL can be exposed via their library [TODO] as well as the CLI [we will not use the CLI when used inter-library rather than as a direct CLI dependency], we would thus depend on its optional gh integration (i.e. not remake this but would need to retain the documentation of the expectation for how to use it for auth)"

**A5.** The spec mentions `polars-ls` and `octopolars` - are these existing packages or conceptual components to be developed?

**Your answer:**
"polars-ls is a simple way to get files from the file system as dataframes (given a local path like how octopolars gets them from the names of github repos). these packages already exist and will be included as dependencies. polars-ls will be a dep, octopolars will be an optional dep (do not enforce dependency on the github integration, not necessary for an MVP nor for basic usage and URL access will always be slower than accessing already existing local files on disk)."

**A6.** For the LLM integration:
- Will users need to provide their own API keys or will there be a default model?
- Will there be local model options for privacy-sensitive codebases?
- What are the cost implications for users?

**Your answer:**
"we will get to that bridge when we cross it, not implemented yet, but general idea will be bring your own API key yes. I would like to permit usage of local models, but again we will operate on a minimal software dependency basis (we don't want to enforce heavyweight dependencies besides basic dataframe parsing). it should be possible to use local LLMs as well as remote APIs though. (note that it can also be desirable to deploy local LLMs via servers too so these modes arent necessarily independent). yes of course APIs will have cost implications."

**A7.** For embeddings:
- Is `polars-fastembed` an existing package or something to be developed?
- Will there be different embedding model options?
- What are the storage requirements for typical codebases?

**Your answer:**
"polars-fastembed is an existing package which computes vector embeddings directly on polars dataframes. again, we offload the complexity of this to that package. this pattern lets us keep our software nice and simple, just composing independent parts. polars-fastembed will be made a default dependency, in this case I expect polars-fastembed to be more desirable than API usage (fast to compute and simple to implement thus competitive to commercial offerings). storage requirements unknown initially but these vector embeddings are compact."

**A8.** The `octopolars` CLI had a default subcommand (`repos`). Should `ntnts` have a default action?

**Your answer:**
"we want a 'split' CLI, the purpose of the CLI will be multiple commands: one will be retrieve, one will be embed so you would call it like `ntnts retrieve . --query "I want to modify the database"` to retrieve the parts of the codebase pertaining to DB modifications, or a command like `ntnts embed --rerank . --query "neural networks"` would produce a Dataframe of the files sorted by similarity to the query (thus the head of the df would surface the neural net related code files) and so on. EDIT: actually no let's just do one command. less is more when it comes to CLI complexity."

**A9.** Will the subcommands follow a strict sequential workflow (scan → analyze → embed) or can they be used independently?

**Your answer:**
"the commands will be distinct and will share a syntax. let's start by just doing embed and retrieve. EDIT: actually you can do them all as one command `ntnts` for brevity and simplicity, retrieval can just be controlled by a flag indicating the `k` to retrieve. less is more when it comes to CLI complexity."

**A10.** Should commands support piping between them (e.g., `ntnts scan ... | ntnts analyze`)?

**Your answer:**
"the commands will allow outputting JSON so they will be pipeable but the idea is not to pipe from the tool back into itself (re-computing embeddings) that would be silly."

**A11.** What's the expected user workflow? Will they typically run all commands in sequence or use specific features independently?

**Your answer:**
"the user will run the command and get the dataframe on the CLI, and then when they're happy with it they'll either output it or just use the info (or go put an equivalent library code usage in their code)"

**A12.** What kinds of repositories/codebases is this tool optimized for? (size, language, structure)

**Your answer:**
"we shall see, but no reason to presume small repos only, fastembed is fast!"

**A13.** How will the tool handle very large codebases? Will it need pagination, sampling, or other optimization strategies?

**Your answer:**
"do not get ahead of ourselves with advanced ideation. we'll optimise as best we can."

**A14.** What is the primary output format users will want? (Parquet seems default in spec, but should JSON, CSV, etc. be supported?)

**Your answer:**
"primarily we will output to the terminal and control other outputs (JSON, parquet etc) via flags. in library code of course that just equates to returning a dataframe, which is often the most useful."

**A15.** Are there specific integrations with IDEs or other developer tools planned?

**Your answer:**
"No IDE integration, this is a standalone tool."

**A16.** How should the tool handle partial or incomplete analysis (e.g., if some files can't be parsed)?

**Your answer:**
"since the tool offloads its file reading to polars-ls or octopolars we dont need to worry about IO hassle"

**A17.** What are the exact Python version requirements?

**Your answer:**
"python version requirements TBD, probably 3.9+ or 3.10+ (the readme will get this automatically via the badge it gets from PyPI classifiers we dont need to include that)"

**A18.** Will installation extras be needed (similar to octopolars' `[polars]` extras)?

**Your answer:**
"yes, polars or polars-lts-cpu will need to be chosen, like in octopolars and polars-fastembed"

**A19.** Are there non-Python dependencies that users will need to install separately?

**Your answer:**
"no non-python dependencies are needed to install. polars-fastembed is python and rust based but we ship the rust part in a wheel to pypi. linux, mac and windows are all supported"

**A20.** What level of technical knowledge should the README assume?

**Your answer:**
"the readme should be similar in style to the original one"

**A21.** What kind of example repositories would best showcase the tool's capabilities?

**Your answer:**
"dont get ahead of ourselves with example repos"

**A22.** Should the README include performance benchmarks or expected resource usage?

**Your answer:**
"dont get ahead of ourselves with perf benchmarks - we have nothing to compare to. this will itself be a good showcase of polars-fastembed usage in practice"

**A23.** Will this be under the same MIT license as octopolars?

**Your answer:**
"yes MIT license"

**A24.** Are there any commercial usage restrictions, especially regarding the LLM integration?

**Your answer:**
"no commercial usage restrictions from MIT license etc"

**A25.** Who will maintain the project, and what's the contribution model?

**Your answer:**
"same maintainer, me"

**A26.** Among the "Future Roadmap" items, which are priorities for documentation in the initial README?

**Your answer:**
"the future roadmap is more like "ideas" than roadmap. the qdrant and lancedb seem worth pursuing but not in MVP. merging multi-repo aggregation is a great idea which would let you "mix repos""

**A27.** Should the README include warnings about the experimental nature of any features?

**Your answer:**
"no dont try to dissuade people from using it by using pejoratives, theres no harm in using dataframe libraries"

**A28.** What is the expected timeline for moving from MVP to more advanced features?

**Your answer:**
"dont commit to any timeline, its in development now"

**A29.** How should the README address the security concerns around sending code to LLMs?

**Your answer:**
"dont comment on security aspects, as LLM APIs are not used by default"

**A30.** Are there specific compliance requirements for different industries that should be mentioned?

**Your answer:**
"dont comment on compliance (not my department)"

## Interview B

**B1.** What does `ntnts` stand for or mean? Understanding the name would help frame the introduction.

**Your answer:**
"ntnts should be introduced with `(pronounced "intents")` and this of course pertains to the idea of obtaining the intents of a repo in a CLI like way, as discussed"

**B2.** Regarding the core functionality:
- You mentioned intents (typically 5) that get condensed into a vector. Should the README specify this 5-intent approach or keep it more flexible?
- Should we emphasize the MutaGReP influence explicitly or just describe the functionality?

**Your answer:**
"no you should mention (default: 5) because it helps give a concrete idea of what we're talking about. homage to mutagrep is a great idea, it's related work - sort of a reimplementation/reproduction of the work, but may expand to go further or in a different direction (no specific guarantee)"

**B3.** For the core command structure:
- You mentioned initially wanting split commands, then suggested just one command. Can you confirm the preferred command structure?
- What parameters will be most important for the primary command(s)?

**Your answer:**
"I am going to go with one command, it's cleaner. thanks for clarifying. we don't need to flesh it all out yet what the CLI args will be. go with what i've mentioned, it's subject to change."

**B4.** Are there specific output formats to highlight besides JSON and terminal display?

**Your answer:**
"we can cover specific output formats later, JSON and terminal will be the main ones"

**B5.** For the implementation:
- How should symbol extraction work? (AST parsing, regex, or other approach)
- Will there be language-specific considerations to mention?

**Your answer:**
"we don't need to detail the symbol mining process in the readme (yet). nor language specific considerations."

**B6.** From a user perspective, what would be a simple example workflow to include in the README?

**Your answer:**
"we will add examples later."

**B7.** Are there any special installation considerations beyond the polars extras?

**Your answer:**
"no special installation considerations besides the polars extras"

**B8.** What's the minimum viable set of features to document for the initial release?

**Your answer:**
"MVP for initial release is use on a local directory of source code, as described"