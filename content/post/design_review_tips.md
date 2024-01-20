+++
title =  "8 Tips for Effective Design Reviews"
date =  2024-01-06T11:07:55-05:00
menu = "main"
+++

### 8 Tips for Effective Design Reviews

I had a recent design review that caused me to self-reflect and come up with some practical technical and non-technical tips on some aspects that may be overlooked in design reviews (lower-level designs in particular).

### Technical Tips

#### 1. The S in SOLID

In my lens, making code changes comes at the cost of introducing complexity. In order to reduce the complexity, I tend to lean towards modifying existing classes. I've learned that this is a bad practice and leads to classes being overloaded, diminishing readability and going against the single responsibility principle.

- **Tip:** 👉 Every class should serve a single purpose. If you are changing that purpose, either create a new class or rename the existing one. Same goes for method names, if you're thinking of changing the return type, think about changing the method name.

#### 2. Dead Code

Sometimes when I am in the design or planning phase, I get so caught up in the new methods that need to be implemented that I forget to mention what will be removed/refactored. Of course, this doesn't hinder the development but it can cause confusion. As I was introducing new methods, my colleagues were left wondering what would happen to the existing ones.

- **Tip:** 👉 Consider additions **and removals** during bug fixes or improvements. Clearly communicate changes to avoid confusion among colleagues.

#### 3. Read-Write Consistency

In my case, I was thinking of writing to a DynamoDB and then reading the value that I just wrote. DynamoDB does not guarantee read after write consistency but rather eventual consistency. So in some cases, I would be reading from stale data. This was a miss in my design and I also lost an opportunity to optimize by cutting down on database calls. 

- **Tip:** 👉 Prioritize considerations of read-after-write consistency, as overlooking this can lead to reading from potentially stale data.

#### 4. Stubbing

Despite going through the design review, there was still the question of who is going to work on what and how we can distribute tasks and minimize conflicts. My colleagues suggested stubbing - adding mocked implementations/signatures so devs know where a change should go.

- **Tip:** 👉 Incorporate stubbing to provide a visual representation of necessary modifications, empowering colleagues to make independent contributions to the system.

### Non-Technical Tips

#### 5. Always Provide Options

A lot of the confusion and miscommunication came from a result of me not providing the different options. It felt like I was suggesting something and my colleagues were opposing me, whereas, in my head, I had already compared pros and cons of possible solutions and presented the 'best' one (in my opinion). There are two problems with this - one, it looks like I never considered other options and two, the option I presented might not even be the best one! When it comes to high-level designs, we have been trained to present multiple options, but with low-level designs, I forget since it is so close to the implementation phase.

- **Tip:** 👉 Present multiple solutions with pros and cons for informed decision-making to avoid overlooking alternatives.

#### 6. Don't Assume

Since I was working on a code base that I had less experience with and my colleagues regularly committed to, I made the assumption that they would know the ins and outs of the code. This is obviously wrong. If it were me who had a lot of experience in the codebase, there would still be minor things here and there that I had no idea about. 

- **Tip:** 👉 Don't assume someone's level of knowledge of a topic at hand; always provide necessary context for a shared understanding.

#### 7. Effective Communication

For me, it's hard to follow when people are talking. I am more of a visual or hands-on learner. To combat this, I try to write down what I hear. When I was chatting with my colleagues and sharing screen, I found it difficult to keep up and note down things at the same time. This shows I need to practice my note-taking in real-time and build up my comprehension.  

- **Tip:** 👉 Recognize diverse learning styles; practice real-time note-taking during dynamic discussions for better comprehension.

#### 8. Knowing When to Stop

I noticed after the 1 hour mark the meeting tone went from productive to slightly negative, leaving me feeling a bit down. Since it was an adhoc meeting, I should have done regular time-checks and ended the meeting at the 1 hour mark (or even earlier). Ideally it should be a scheduled meeting with a clear agenda, but we were on strict timelines.

- **Tip:** 👉 Be mindful of meeting tone; set time limits to maintain productive discussion and a positive atmosphere.

These tips aim to guide a more effective design review process by addressing commonly forgotten elements.