# TakeMeter Planning Document

## Community

I chose student study discussions from public online communities because students often share many different types of comments: study advice, complaints, motivation requests, and questions about resources. This makes the community a good fit for a text classification task because the discourse is varied and meaningful.

## Labels

### study_strategy
A comment that gives a clear study method, technique, routine, or practical advice that could help someone study better.

### complaint
A comment that mainly expresses frustration, stress, disappointment, or dissatisfaction about school, studying, exams, or learning.

### motivation_request
A comment where the writer is asking for encouragement, discipline help, accountability, or support to keep going.

### resource_question
A comment asking for tools, apps, notes, videos, PDFs, books, websites, or other study resources.

## Hard Edge Cases

Some comments may include both a complaint and a request for help. For example: “I keep failing my quizzes and I don’t know what app can help me study better.”  
Decision rule: If the main purpose is asking for a tool or material, label it resource_question. If the main purpose is expressing frustration without asking for a specific resource, label it complaint.

Some comments may include both motivation and strategy. For example: “I have no motivation, but I’m trying Pomodoro now.”  
Decision rule: If the comment mainly asks for emotional support, label it motivation_request. If it mainly explains a technique, label it study_strategy.

## Data Collection Plan

I will collect at least 200 public student study-related posts or comments. I will aim for a balanced dataset with around 50 examples per label. If one label has too few examples, I will collect more examples specifically for that label.

## Evaluation Metrics

I will use accuracy to measure the overall number of correct predictions. I will also use precision, recall, and F1 score for each label because accuracy alone may hide cases where the model performs badly on one class.

## Definition of Success

A useful classifier should achieve at least 70% overall accuracy and should not completely fail on any label. I will consider the model good enough if most per-class F1 scores are around 0.65 or higher and the mistakes are understandable from the text.

## AI Tool Plan

I will use AI to stress-test my label definitions by generating examples that sit between two labels. I may use AI to pre-label some examples, but I will manually review every label before using it in training. After evaluation, I will use AI to help identify patterns in wrong predictions, then verify those patterns myself.



##Comment Samples
text,label,notes
"I started reading my lecture notes out loud and then explaining each section like I was teaching a classmate. It showed me exactly where I was confused.",study_strategy,"Explains a concrete study method"
"I studied all weekend and the exam still felt like it came from a completely different class.",complaint,"Frustration about exam mismatch"
"How do you make yourself start studying when you already feel behind?",motivation_request,"Asks for motivation help"
"Does anyone know a free app that can turn notes into flashcards?",resource_question,"Asks for study tool"

#Labels

study_strategy
complaint
motivation_request
resource_question