# The SLING task system

The SLING task system allows you to define a workflow consisting of tasks that
you want executed. You can define dependencies between tasks and set up
channels so the tasks can communicate with each other using messages.

## Component-based architecture
All in all, the task system allows you to define workflows including the data-
and control flow with the set of tasks that you want executed to get your job
done. I.e., the SLING task system provides a sound basis for building your
system using a component-based architecture.

## Drive execution from Python
While written in C++, the task system also has a Python interface which
provides the additional benefit of being able to drive a full workflow
including, e.g., input file processing, training, etc. all written in C++,
from a simple Python script. (See example below).

## Monitoring a running workflow
Once a workflow is running, you have access to easy monitoring of the current
running tasks in your browser. By default, you can access the web server
exposing the monitoring on http://localhost:6767
<TO-DO add screenshot>

## Workflow example

The following example shows a simple workflow in the form of a parser trainer.
It consists of a single task with three inputs and one output defined. The
three inputs are the training set, dev set and word embeddings and the output
is the model with the trained weights.  

```
import sling
import sling.task.workflow as workflow

# Start up workflow system.
workflow.startup()         

# Create workflow and name it “parser-training”.
wf = workflow.Workflow("parser-training") 

# Set input training data
training_corpus = wf.resource(
  "local/data/corpora/caspar/train_shuffled.rec",
  format="record/document"
)

# Set input eval data
evaluation_corpus = wf.resource(
  "local/data/corpora/caspar/dev.rec",
  format="record/document"
)

# Set input word embeddings 
word_embeddings = wf.resource(
  "local/data/corpora/caspar/word2vec-32-embeddings.bin",
  format="embeddings"
)

# Set the output parser model
parser_model = wf.resource(
  "local/data/e/caspar/caspar.flow",
  format="flow"
)

# Create the parser trainer task.
trainer = wf.task("caspar-trainer")

# Define task parameters
trainer.add_params({
  "learning_rate": 1.0,
  "learning_rate_decay": 0.8,
  "clipping": 1,
  "optimizer": "sgd",
  "epochs": 50000,
  "batch_size": 32,
  "rampup": 120,
  "report_interval": 500
})

# Attach the defined input and output resources to the trainer task 
trainer.attach_input("training_corpus", training_corpus)
trainer.attach_input("evaluation_corpus", evaluation_corpus)
trainer.attach_input("word_embeddings", word_embeddings)
trainer.attach_output("model", parser_model)

# Run parser trainer.
workflow.run(wf)

# Shut down.
workflow.shutdown()
```
