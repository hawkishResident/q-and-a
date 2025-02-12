# Technical Interview Flow Guide (30-40 minutes)

## Opening (10 minutes)
Start with core technical assessment to quickly gauge expertise level.

### 1. Node.js and Express.js Core
"Could you explain what Node.js is and a bit about the event loop?"
**[@ref: What is Node.js]**
**[@ref: Event Loop, Micro/Macro Tasks]**
- Quick follow-up about micro/macro tasks if they show strong knowledge
- Transition: "Talking about Node.js, could you implement a middleware in Express that calculates request processing time?"**[@ref: What is Middleware]**

Here are several approaches to implementing request time middleware, from basic to advanced:

1. Two-Middleware Solution (Shows better design understanding):
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

2. Single Middleware Solution (Basic approach):
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

2. Event-Based Solution (More complete with multiple events):
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

3. Advanced Solution (Shows deeper understanding):
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
- Understanding of middleware concept
- Proper use of next()
- Handling response events
- Timing accuracy consideration
- Error handling awareness **[@ref: Design Patterns in Node.js]**

- This shows practical coding skills and understanding of async concepts

## Mid-Level Technical (15 minutes)

### 2. Database and Architecture
"What's your experience with databases? Could you compare MongoDB and PostgreSQL?"
**[@ref: MongoDB vs PostgreSQL]**
- Quick Redis discussion if they have experience, power off scenario
**[@ref: Redis]**
- "Can you explain the CAP theorem and how it applies to different databases?"
**[@ref: CAP Theorem]**
- "What role do indexes play in database performance?"
**[@ref: Database Indexing]**
- Natural transition to architecture: "How did you handle communication between services?"
**[@ref: Microservices vs Monolith]**
- If they mention message queues, briefly discuss RabbitMQ/Kafka
**[@ref: RabbitMQ vs Kafka]**

### 3. Containerization and Deployment
"How do you typically deploy your Node.js applications?"
**[@ref: What is Docker]**
**[@ref: Containerization]**
- Docker and K8s experience
**[@ref: Container Orchestration]**
- "Could you explain GitOps and its principles?"
**[@ref: GitOps]**
- Scaling strategies **[@ref: Strategies]**
- Brief AWS services discussion
**[@ref: AWS Services with K8s]**
- CI/CD/CD pipeline overview
**[@ref: CI/CD/CD]**

## Closing Section (10 minutes)

### 4. System Design AND Technical Achievement
Choose either:
- Your prepared system design question about async communication
AND
- "Tell me about a technical achievement you're proud of"


### 5. Quick Technical Deep-Dive (if time permits)
Based on their previous answers, choose one:
- Big O notation and complexity **[@ref: Common Complexities]**
- TLS and networking **[@ref: TLS Working]**

## Time Management Tips
- Opening (Node.js + Express): 10 minutes
- Databases and Architecture: 7-8 minutes
- Containerization and Deployment: 7-8 minutes
- System Design/Achievement: 7-8 minutes
- Final Deep-Dive: 3-5 minutes
- Buffer time: 2-3 minutes

## Priority Topics to Cover
1. Node.js fundamentals (event loop, middleware)
2. Database experience
3. Service communication
4. Deployment approach
5. One significant achievement or system design

## Notes for Interviewer
- Keep initial answers focused - redirect if they go too deep into any topic
- Skip follow-up questions unless they're crucial
- Have your system design question ready
- Look for:
  - Practical experience vs theoretical knowledge
  - Problem-solving approach
  - Communication clarity
  - Understanding of trade-offs