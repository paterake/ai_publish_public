Give an LLM 31 columns and ask “what’s interesting?” and it will generate noise.
Wide schemas explode the hypothesis space.
So I scope columns per question before the model sees the data summary.

Giving an LLM a 31-column dataset and asking "what is interesting?" is a reasonable question with an unreasonable answer surface.

The model generates hypotheses. Some are useful. Most are plausible combinations of columns that produce charts no one needed — because with 31 columns, the combination space is large and the model cannot know which subset maps to your actual business question.

The fix is column scoping: you specify which columns are relevant to the question before the LLM sees the data summary. The model reasons within a constrained space — not because it has been told what to conclude, but because it has been given the right evidence set.

For a 31-column incident dataset, three analytical questions map to three column sets:

- Taxonomy health → category, subcategory, priority, close_code
- Operational routing → assignment_group, sla_breach, reopen_count
- Data quality → state, close_code, resolution_notes, reopen_count

📌 Three focused runs instead of one broad run. Each addresses one question clearly.

Secondary benefit: fewer hallucinated column names. With 4-6 columns in scope instead of 31, the model has fewer opportunities to invent a field it "remembers" from similar datasets.

Article linked below — Part 3 of my series on local-first AI visualisation.

Do you constrain the question you give to AI tools, or ask broad and curate afterward?

#DataVisualization #DataEngineering #AI #Python #LLMOps
