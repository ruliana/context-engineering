# Blog Writing Context

## Key Concepts

- **E-E-A-T**: Experience, Expertise, Authoritativeness, Trustworthiness - Google's framework for evaluating content quality
- **Technical Blog**: Informal, conversational content that explains technical concepts, solutions, or processes using "you", "I", and "we" pronouns
- **Dwell Time**: Average engagement time per session, indicating content quality and reader satisfaction
- **Technical How-to**: Step-by-step guides that solve specific technical problems or demonstrate implementations
- **Content Scope**: Clearly defined boundaries of what the article covers and explicitly what it doesn't cover
- **Prerequisites**: Required knowledge or tools readers need before starting the content
- **DACI Framework**: Driver, Approver, Contributors, Informed - decision-making structure for content planning
- **Bounce Rate**: Percentage of unengaged sessions (less than 10 seconds, no conversion, single pageview)
- **Click-through Rate (CTR)**: Percentage of readers who click on links or CTAs out of total viewers
- **Engagement Session**: Session lasting longer than 10 seconds, with conversion event, or 2+ pageviews

## Common Patterns

### Problem-Solution Structure
```markdown
# [Specific Problem] - [Brief Solution Preview]

## The Problem
- Context: What situation triggers this need
- Impact: Why this matters technically/business-wise
- Current limitation: What doesn't work with existing approaches

## The Solution
[High-level approach in 1-2 sentences]

## Prerequisites
- [Tool/knowledge requirement 1]
- [Tool/knowledge requirement 2] 
- [Tool/knowledge requirement 3]

## Implementation
[Step-by-step technical guidance]

## Validation
[How to verify the solution works]

## What's Next
[Optional follow-up actions or related topics]
```
**When to use**: Solving specific technical challenges or demonstrating new approaches
**Expected output**: Clear problem statement followed by actionable solution

### Tutorial/Guide Structure
```markdown
# How to [Accomplish Specific Task]

## What You'll Build
[Brief description of end result]

## Prerequisites
- [Required knowledge/tools]

## Step 1: [Action Verb] [Specific Task]
[Exact commands/code with explanations]

## Step 2: [Action Verb] [Specific Task]
[Exact commands/code with explanations]

## Step 3: [Action Verb] [Specific Task]
[Exact commands/code with explanations]

## Testing Your Implementation
[Validation steps]

## Troubleshooting
[Common issues and solutions]

## Conclusion
[Summary and next steps]
```
**When to use**: Teaching readers to build or configure something step-by-step
**Expected output**: Working implementation following the guide

### Deep Dive/Analysis Structure
```markdown
# Understanding [Technical Concept]: [Key Insight]

## Why This Matters
[Business/technical relevance]

## Core Concepts
[Essential definitions and principles]

## How It Works
[Technical explanation with examples]

## Practical Applications
[Real-world use cases]

## Performance Considerations
[Trade-offs, limitations, optimizations]

## Best Practices
[Recommended approaches]

## Further Reading
[Authoritative sources only]
```
**When to use**: Explaining complex technical concepts or providing technical analysis
**Expected output**: Comprehensive understanding of the topic

### Migration/Comparison Structure
```markdown
# Migrating from [Technology A] to [Technology B]: [Key Benefits]

## Why Consider Migration
[Business and technical drivers]

## Key Differences
| Feature | Technology A | Technology B |
|---------|-------------|-------------|
| [Aspect] | [Details] | [Details] |

## Migration Strategy
[High-level approach]

## Step-by-Step Migration
[Detailed technical steps]

## Validation and Testing
[How to verify successful migration]

## Rollback Plan
[How to revert if needed]

## Performance Comparison
[Before/after metrics]
```
**When to use**: Helping readers evaluate and execute technology transitions
**Expected output**: Clear migration path with risk mitigation

## Implementation Details

### Content Planning and Research
1. **Define Scope and Non-Scope**
   ```markdown
   ## What This Article Covers
   - [Specific topic 1]
   - [Specific topic 2]
   - [Specific topic 3]
   
   ## What This Article Doesn't Cover
   - [Related but excluded topic 1]
   - [Related but excluded topic 2]
   ```

2. **Identify Prerequisites**
   ```markdown
   ## Prerequisites
   This article assumes you are familiar with:
   - [Technology/concept 1] - [Brief explanation if needed]
   - [Technology/concept 2] - [Brief explanation if needed]
   - [Required tools/access]
   ```

3. **Research Validation**
   - Check publication date (prefer recent sources)
   - Verify author credentials and organization authority
   - Confirm primary source citations
   - Test all code examples and commands

### Writing and Structure

1. **Opening Hook (First 2-3 sentences)**
   ```markdown
   [Specific problem statement or compelling use case]
   [Brief preview of solution/approach]
   [Article scope in one sentence]
   ```

2. **Scannable Content Format**
   - Use short sentences (15-20 words maximum)
   - Write short paragraphs (3-4 sentences)
   - Include bullet points for lists
   - Add code blocks with syntax highlighting
   - Use subheadings every 200-300 words

3. **Code Example Standards**
   ```markdown
   ### [Descriptive Heading]
   ```[language]
   [Complete, working code example]
   ```
   
   **What this does**: [Brief explanation]
   **Expected output**: [What success looks like]
   **Common errors**: [Typical issues and solutions]
   ```

4. **Visual Elements Integration**
   - Architecture diagrams for system overviews
   - Screenshots for UI-based instructions
   - Code snippets with clear annotations
   - Tables for feature comparisons
   - Flowcharts for decision processes

### Content Optimization

1. **SEO and Readability**
   - Primary keyword in title and first paragraph
   - Descriptive subheadings with keywords
   - Internal links to related content
   - External links to authoritative sources only
   - Alt text for all images

2. **Technical Accuracy**
   - Test all code examples in clean environment
   - Verify version compatibility
   - Include error handling examples
   - Document expected outputs
   - Provide troubleshooting guidance

3. **Accessibility Standards**
   - Use semantic HTML structure
   - Provide alt text for images
   - Ensure sufficient color contrast
   - Write descriptive link text
   - Structure content with proper headings

## Validation Methods

### Pre-Publication Validation

1. **Technical Accuracy Check**
   - [ ] All code examples tested and working
   - [ ] Commands executed in clean environment
   - [ ] Error scenarios documented
   - [ ] Version compatibility verified
   - [ ] Prerequisites clearly stated

2. **Content Quality Assessment**
   - [ ] Scope clearly defined
   - [ ] Non-scope explicitly stated
   - [ ] Problem-solution alignment clear
   - [ ] Step-by-step instructions complete
   - [ ] Validation steps provided

3. **Readability Validation**
   - [ ] Average sentence length under 20 words
   - [ ] Paragraph length 3-4 sentences
   - [ ] Subheadings every 200-300 words
   - [ ] Technical jargon defined
   - [ ] Code examples properly formatted

4. **Authority and Source Check**
   - [ ] All sources are primary/official documentation
   - [ ] External links lead to authoritative sources
   - [ ] Claims backed by credible references
   - [ ] Author expertise clearly established
   - [ ] Publication date current

### Post-Publication Metrics

1. **Engagement Metrics**
   - **Target**: Engagement sessions > 60% (sessions lasting 10+ seconds, conversions, or 2+ pageviews)
   - **Target**: Bounce rate < 40%
   - **Target**: Average engagement time > 3 minutes
   - **Target**: Click-through rate on CTAs > 2%

2. **Content Performance Indicators**
   - **Page views**: Traffic volume and growth trends
   - **Time on page**: Indicates content depth and value
   - **Scroll depth**: Shows content consumption patterns
   - **Social shares**: Indicates content resonance
   - **Comments/feedback**: Shows community engagement

3. **Technical Performance**
   - **Page load speed**: Target < 3 seconds
   - **Core Web Vitals**: Meet Google's performance standards
   - **Mobile responsiveness**: Proper display across devices
   - **Accessibility score**: WCAG compliance

4. **Search Performance**
   - **Organic traffic**: Growth in search visibility
   - **Keyword rankings**: Position for target terms
   - **Featured snippets**: Structured data optimization
   - **Backlinks**: External site references

### Content Quality Framework (E-E-A-T)

1. **Experience Validation**
   - [ ] Author has hands-on experience with the technology
   - [ ] Examples based on real-world implementations
   - [ ] Practical insights beyond basic documentation
   - [ ] Lessons learned from actual usage

2. **Expertise Assessment**
   - [ ] Technical accuracy verified by subject matter experts
   - [ ] Depth of knowledge demonstrated through examples
   - [ ] Complex concepts explained clearly
   - [ ] Advanced use cases covered appropriately

3. **Authoritativeness Check**
   - [ ] Author credentials clearly stated
   - [ ] Organization/company authority established
   - [ ] References to authoritative sources
   - [ ] Recognition within technical community

4. **Trustworthiness Verification**
   - [ ] Transparent about limitations and trade-offs
   - [ ] Acknowledges when approaches may not work
   - [ ] Provides alternative solutions
   - [ ] Updates content when information changes

### Decision Framework for Content Planning

1. **Content Type Selection**
   ```
   IF (specific problem + solution exists)
       THEN use Problem-Solution Structure
   ELSE IF (teaching step-by-step process)
       THEN use Tutorial/Guide Structure  
   ELSE IF (explaining complex concept)
       THEN use Deep Dive/Analysis Structure
   ELSE IF (comparing technologies)
       THEN use Migration/Comparison Structure
   ```

2. **Scope Definition Decision**
   ```
   IF (topic can be covered comprehensively in 1500-3000 words)
       THEN create single comprehensive article
   ELSE IF (topic requires multiple complex sections)
       THEN create article series with clear navigation
   ELSE IF (topic is too narrow)
       THEN combine with related topics
   ```

3. **Audience Level Assessment**
   ```
   IF (beginners to intermediate)
       THEN include more background context and definitions
   ELSE IF (intermediate to advanced)
       THEN focus on implementation details and best practices
   ELSE IF (expert level)
       THEN emphasize performance, edge cases, and optimization
   ```

## Authoritative References

- [Google Developer Documentation Style Guide](https://developers.google.com/style) - Editorial guidelines for technical documentation
- [Google Technical Writing Courses](https://developers.google.com/tech-writing) - Technical writing fundamentals and best practices
- [Google Search Quality Guidelines](https://developers.google.com/search/docs/fundamentals/creating-helpful-content) - Content quality standards and E-E-A-T framework
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/) - Technical how-to posts and architecture guidance
- [Microsoft Writing Style Guide](https://docs.microsoft.com/en-us/style-guide/) - Brand voice principles and accessibility guidelines
- [GitHub Technical Writing Template](https://github.com/BolajiAyodeji/technical-writing-template) - Sample template with guidelines for technical articles
- [Atlassian DACI Decision Framework](https://www.atlassian.com/software/confluence/templates/decision) - Decision-making structure for content planning
- [Diátaxis Documentation Framework](https://diataxis.fr/) - Systematic approach to organizing technical documentation