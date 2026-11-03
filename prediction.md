Written before running baseline.py or looking at any Unit 6 numbers.

The baseline will beat my trained model on `question` and probably on `evidence`, because "is this post a question" and "does it contain a link or number" are things a general model already knows, and my trained model sat at F1 0.000 on `question`. My trained model will beat it on `opinion` and `experience` (the labels the zero-shot model has no good way to tell apart), but I expect the baseline to be close to or better than the trained model overall, since I only have 152 training posts.
