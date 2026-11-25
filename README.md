# Gen-LM

Generative Language Model (GLM) is a research project dedicated to creating a generative language model based on a dynamically growing concept graph.

Unlike traditional LLMs (Transformers), which rely on fixed topologies and gradient descent, Gen-LM explores an alternative approach:

Merge: Semantically close nodes merge into a higher-level concept (energy+1).

Growth: New nodes are born from context conflicts.

Shatter: Frozen concepts can be shattered back into their components.

All simulators run directly in the browser using Transformers.js to load real neural network embeddings (MiniLM, NLLB-200), requiring no backend or server-side processing.

📂 Project Structure (/HTML5)
Each file is a standalone interactive experiment. Just open the desired .html file in your browser (or navigate via GitHub Pages).

1. AI_translate.html — Semantic Translator
What it does: Translates text from one language to another (supporting 200+ languages via NLLB-200) and displays the "semantic similarity" between the original and the translation as a percentage.
Why it matters: Demonstrates how embeddings can evaluate translation quality without a reference text, while also providing a practical translation tool.

2. Black Box Puzzle.html — "Black Box" Puzzle Game
What it does: A turn-based puzzle. The player is given two words (e.g., apple and banana) and must find a bridge word (e.g., fruit) to make the neural network merge them into a single concept.
Why it matters: Playfully demonstrates how semantic similarity works and why not all words act as valid "bridges" between concepts.

3. Feed the Brain.html — Graph Growth Simulator
What it does: Allows you to input words or phrases and watch them merge into concepts (E1, E2, E3) based on a similarity threshold.
Why it matters: A visual representation of the Merge mechanism and the birth of abstractions. You can adjust the threshold slider to see how the graph's structure changes.

4. Multilingual Sandbox.html — Multilingual Sandbox
What it does: Shows how the same entity in different languages (e.g., apple, яблоко, pomme) merges into a single node.
Why it matters: Demonstrates that languages are just different shells for the same meanings (concepts). Useful for vocabulary learning and understanding how multilingual models work.

5. game.html and game_2.html — Early Prototypes
What it does: Simpler versions of the interactive graphs (with physics and activation waves).
Why it matters: Kept as the project's development history and for comparison with later, more advanced versions.

6. SemanticChain.html — Semantic Chain (Game)
What it does: A step-by-step puzzle inspired by "Six Degrees of Separation". The player is given two semantically distant words (e.g., moon and banana) and must connect them by entering intermediate bridge words. Each new word must be semantically close to the previous one (similarity threshold > 0.50), and the final word must be close to the target.
Why it matters: It demonstrates how a neural network builds non-obvious semantic links between concepts that have nothing to do with each other in real life. The game supports both Russian and English (RU/EN) and uses a multilingual model, allowing players to input words in either language. It's a perfect example of a "black box": the player has to empirically guess which associations the model considers valid.

🛠️ Technologies
HTML5 / CSS3 / JavaScript (ES6 Modules)

Transformers.js (Hugging Face) — running models (Xenova/all-MiniLM-L6-v2, Xenova/nllb-200-distilled-600M) directly in the browser.

Canvas API — graph visualization and node physics.

#Quick Start:
https://github.com/mrDarkDuck/Gen-LM/blob/main/HTML5/README.md
