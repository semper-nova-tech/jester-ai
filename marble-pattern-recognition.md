# Marble pattern recognition

Product brief. A person adds marbles one at a time. A decision model and a Markov table each guess the next color. The person chooses the pattern, and the pattern decides who is ahead.

This demonstrates a small predictive decision model. The person is the live input. The product does not grade the person.

## Colors

Four colors in this version: red, blue, green, and yellow.

Every guess is a probability for each color, and the four probabilities sum to 1. Adding a color later means new training data and a new last layer. This version stays at four.

## Session

A session starts with an empty sequence.

The person adds one marble at a time. Before each marble is added, both predictors commit their four probabilities from the sequence so far. The marble just added is the truth for that step. Both predictors are scored. The decision model is then revised from that marble. The Markov table increments its counts.

A new session puts the decision model back on its trained prior and empties the Markov table.

## Decision model

The decision model is trained before anyone plays, on the synthetic sequences below. It reads the whole sequence so far and returns a probability for each of the four colors. Those probabilities come from learned weights.

During a session it gets smarter by a Bayesian update on itself.

The weights that read the sequence stay fixed. The last layer, which turns that reading into the four color probabilities, carries a belief: a center and a spread. The center starts at the trained weights. The spread starts wide enough that the first marbles can move it.

Each new marble moves the center toward weights that would have given the chosen color a higher probability on a sequence like the one so far, and tightens the spread. The next guess uses the new center.

Early marbles move the center more, because the spread is still wide. Later marbles move it less, because the belief has tightened. Older marbles are discounted, so a pattern change later in the session can still move the layer.

The Markov table is not an input to this update.

## Markov table

The Markov table counts what followed the recent marbles in this session. It has two short memories:

- What followed the last color.
- What followed the last pair of colors.

Every cell starts at a count of one, so a color that has not happened yet still has a small probability. Each marble increments the cells for the transition that just happened.

With no marbles yet, all four colors are equal. After one marble, the guess comes from the single-color memory. After two or more, the guess comes from the pair memory.

The table has no trained weights. It only knows what this session has counted inside that short window.

Both predictors see the same sequence and name the same four colors. Their updates stay separate.

## Training data

Train the decision model on synthetic sequences generated from known rules. Each training item is a prefix of a sequence and the color that comes next.

The generator includes all of these:

- A favorite color, held with noise.
- Alternation of two colors.
- Short runs of the same color.
- Longer repeating rules that a window of one or two marbles cannot represent. Include a repeating block of red, red, blue, and a rule where every third marble is one fixed color.
- A switch from one rule to another partway through a sequence.
- Unstructured sequences, where each marble is independent of the ones before it.

### Advisory: synthetic data

The generator is the model's world. A rule left out of it is absent from the prior. The session update can bend the last layer only along patterns the frozen sequence-reading already expresses. A rule the generator never built stays weak across a long session.

If the generator is mostly short habits, the decision model learns the same job as the Markov table, and the comparison goes flat. The longer rules above are what give the decision model a different job.

Synthetic sequences are not people. The live session is the test. Fitting the generator until the model beats the table on generated sequences says nothing about a person.

Hold out at least one longer rule from training and keep it for live sessions. That is how the product shows a pattern the prior has never seen.

Record the noise rate and the switch rate used to generate the data. Those two numbers set how fast either predictor should lock on. Changing them later produces a different model.

## Who is ahead

The person decides who is ahead by the pattern they enter. The product keeps both scores on every marble. It does not name a winner for the session.

**Short habit.** One favorite color, a two-color alternation, or runs of two. After a few repetitions the Markov table matches or passes the decision model. The table is counting that window exactly.

**Longer rule that was in the generator.** The decision model leads from the first cycle. The prior already scores that sequence. The short Markov window cannot represent the rule, so the table stays behind.

**Longer rule held out of training.** The short Markov window stays behind. A long sequence does not, by itself, make the decision model track the rule. The session update helps only when the frozen reading can express that rule. Otherwise the person can keep both predictors missing for the whole session.

**A switch mid-session.** Discounting lets the decision model's last layer follow the new rule. The Markov table follows as fast as its short counts replace the old ones. Which predictor leads after the switch depends on the rule the person switched to.

## Scores

On every marble, for each predictor, record two numbers:

- **Hit.** The color with the highest probability was the marble just added.
- **Assigned probability.** The probability that predictor had placed on the marble just added, before it was added.

Both run for the whole session, on the same marbles. Those two running comparisons are the result. A predictor can lead on hits and trail on assigned probability. Both belong in the result.

## Left out of this brief

Screen layout, controls, and visual design.

Programming language, libraries, and file formats.

A loop that trains the model on its own guesses. The only evidence is the marble the person added.

A blend that mixes the Markov probabilities into the decision model. The comparison depends on the two staying separate.
