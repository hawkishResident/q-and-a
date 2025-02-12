# Technical Interview Flow Guide (30-40 minutes)

## Opening (10 minutes)
Start with core technical assessment to quickly gauge expertise level.

### 1. Node.js and Express.js Core [Junior → Senior]
"Could you explain what Node.js is and a bit about the event loop?"
**[@ref: What is Node.js]** [Junior]
**[@ref: Event Loop, Micro/Macro Tasks]** [Middle → Senior]
- Quick follow-up about micro/macro tasks if they show strong knowledge [Senior]
- Transition: "Talking about Node.js, could you implement a middleware in Express that calculates request processing time?"**[@ref: What is Middleware]** https://js.do/

Here are several approaches to implementing request time middleware, from basic to advanced:

1. Two-Middleware Solution (Shows better design understanding) [Middle]:
```javascript
// First middleware to start the timer
const startTimer = (req, res, next) => {
  req.startTime = Date.now();
  next();
};

// Second middleware to log the duration
const logTime = (req, res, next) => {
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    console.log(`${req.method} ${req.url} took ${duration}ms`);
  });
  next();
};

// Usage:
app.use(startTimer);
app.use(logTime);
```

2. Single Middleware Solution (Basic approach) [Junior]:
```javascript
const basicTimeMiddleware = (req, res, next) => {
  req.startTime = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - req.startTime;
    console.log(`${req.method} ${req.url} took ${duration}ms`);
  });
  
  next();
};
```

3. Event-Based Solution (More complete with multiple events) [Middle → Senior]:
```javascript
const eventTimeMiddleware = (req, res, next) => {
  const startTime = Date.now();
  const timings = {
    total: 0,
    processing: 0
  };

  // Start processing time
  res.on('data', () => {
    if (!timings.processing) {
      timings.processing = Date.now() - startTime;
    }
  });

  // Headers sent
  res.on('headers', () => {
    const headerTime = Date.now() - startTime;
    console.log(`Headers sent in: ${headerTime}ms`);
  });

  // Response finished
  res.on('finish', () => {
    timings.total = Date.now() - startTime;
    console.log(`${req.method} ${req.url} - Processing: ${timings.processing}ms, Total: ${timings.total}ms`);
  });

  // Error handling
  res.on('error', (error) => {
    const errorTime = Date.now() - startTime;
    console.error(`Request error after ${errorTime}ms:`, error);
  });

  next();
};
```

4. Advanced Solution (Shows deeper understanding) [Senior]:
```javascript
const advancedTimeMiddleware = (options = {}) => {
  const {
    slowThreshold = 1000,
    logger = console.log
  } = options;

  return (req, res, next) => {
    req.startTime = process.hrtime();
    
    res.on('finish', () => {
      const [seconds, nanoseconds] = process.hrtime(req.startTime);
      const duration = seconds * 1000 + nanoseconds / 1000000;
      
      const logData = {
        method: req.method,
        url: req.url,
        duration: `${duration.toFixed(2)}ms`,
        status: res.statusCode,
        slow: duration > slowThreshold
      };

      logger(logData);
    });
    
    next();
  };
};

// Usage:
app.use(advancedTimeMiddleware({
  slowThreshold: 500,
  logger: (data) => {
    if (data.slow) {
      console.warn(`SLOW REQUEST: ${JSON.stringify(data)}`);
    } else {
      console.log(`REQUEST: ${JSON.stringify(data)}`);
    }
  }
}));
```

Look for these key points in their solution:
- Understanding of middleware concept [Junior]
- Proper use of next() [Junior]
- Handling response events [Middle]
- Timing accuracy consideration [Middle]
- Error handling awareness **[@ref: Error Handling in Node.js]** [Senior]

## Mid-Level Technical (15 minutes)

### 2. Database and Architecture [Middle → Senior]
"What's your experience with databases? Could you compare MongoDB and PostgreSQL?"
**[@ref: MongoDB vs PostgreSQL]** [Junior → Middle]
- Quick Redis discussion if they have experience, power off scenario [Middle]
**[@ref: Redis]**
- "Can you explain the CAP theorem and how it applies to different databases?" [Senior]
**[@ref: CAP Theorem]**
- "What role do indexes play in database performance?" [Middle → Senior]
**[@ref: Database Indexing]**
- Natural transition to architecture: "How did you handle communication between services?" [Middle → Senior]
**[@ref: Microservices vs Monolith]**
- If they mention message queues, briefly discuss RabbitMQ/Kafka [Senior]
**[@ref: RabbitMQ vs Kafka]**

### 3. Containerization and Deployment [Middle → Senior]
"How do you typically deploy your Node.js applications?"
**[@ref: What is Docker]** [Junior → Middle]
**[@ref: Containerization]** [Middle]
- Docker and K8s experience [Middle → Senior]
**[@ref: Container Orchestration]**
- "Could you explain GitOps and its principles?" [Senior]
**[@ref: GitOps]**
- Scaling strategies **[@ref: Strategies]** [Senior]
- Brief AWS services discussion [Middle → Senior]
**[@ref: AWS Services with K8s]**
- CI/CD/CD pipeline overview [Middle → Senior]
**[@ref: CI/CD/CD]**

## Closing Section (10 minutes)

### 4. System Design AND Technical Achievement [Middle → Senior]
Choose either:
- Your prepared system design question about async communication [Senior]
AND
- "Tell me about a technical achievement you're proud of" [Any Level]

### 5. Quick Technical Deep-Dive (if time permits)
Based on their previous answers, choose one:
- Big O notation and complexity **[@ref: Common Complexities]** [Middle → Senior]
- TLS and networking **[@ref: TLS Working]** [Senior]

## Time Management Tips
- Opening (Node.js + Express): 10 minutes
- Databases and Architecture: 7-8 minutes
- Containerization and Deployment: 7-8 minutes
- System Design/Achievement: 7-8 minutes
- Final Deep-Dive: 3-5 minutes
- Buffer time: 2-3 minutes

## Priority Topics to Cover
1. Node.js fundamentals (event loop, middleware) [Junior → Senior]
2. Database experience [Junior → Senior]
3. Service communication [Middle → Senior]
4. Deployment approach [Middle → Senior]
5. One significant achievement or system design [Any Level]

## Notes for Interviewer
- Keep initial answers focused - redirect if they go too deep into any topic
- Skip follow-up questions unless they're crucial
- Have your system design question ready
- Look for:
  - Practical experience vs theoretical knowledge
  - Problem-solving approach
  - Communication clarity
  - Understanding of trade-offs
  
Evaluation Guide:
Junior Level:
- Solid understanding of Node.js basics
- Can implement basic middleware
- Knows basic database operations
- Basic Docker knowledge

Middle Level:
- Good event loop understanding
- Error handling awareness
- Database optimization knowledge
- Container orchestration familiarity

Senior Level:
- Deep Node.js internals knowledge
- Advanced system design skills
- Complex architecture experience
- Advanced deployment strategies