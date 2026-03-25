# LuminaryAI — Company Introduction

LuminaryAI is an AI-powered marketing personalization platform that serves
real-time product recommendations to 47 e-commerce clients. Their recommendation
engine processes 140 million predictions per day — roughly 1,620 predictions
per second at peak — using machine learning models trained on user behavioral
signals: clicks, views, purchases, dwell time, search queries.

The data science team, led by **Dr. Nadia Osei** (PhD from Cambridge, 12 years
in applied ML), builds models that are technically excellent. The serving
infrastructure, led by **Jeroen van der Berg** (former Netflix infrastructure
engineer), serves predictions at 18ms P99 latency. By any technical measure,
both halves work.

What doesn't work is the seam between them.

Models are trained on historical data. Features are precomputed in batch jobs.
The batch jobs run nightly. By the time a model is serving recommendations at
2 PM on Tuesday, it's working with features that were computed at midnight Monday.
A product that went out of stock at 9 AM Monday is still being recommended at
2 PM Tuesday. A sale that started at 10 AM Tuesday isn't influencing recommendations
at 2 PM Tuesday.

The models are right when they were trained. The features are right when they
were computed. The problem is the gap.

You join as **Senior Backend Engineer** on the ML Infrastructure team — responsible
for the data pipelines that connect the models to the reality they're supposed
to reflect.

"Our biggest clients are starting to ask why we're recommending products that
are sold out," Jeroen tells you on your first day. "Dr. Osei's models don't
make that mistake. Our feature pipeline does."
