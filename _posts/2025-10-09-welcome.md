---
layout: post
title: Welcome to My Learning Journey from project to deploy
date: 2025-10-07 19:00 -0300
tags: [software architecture]
---

## 🎯 Welcome to My Learning Journey

Hello, All !!

I'm excited to share with you a project that's been brewing in my mind for quite some time. This is a comprehensive exploration of how to build **production-ready software** using clean architecture principles, starting from the very first line of code.

Over the next several blog posts, we'll transform a simple YOLO-based car detection system into a robust, scalable application that demonstrates real-world software engineering practices. Each post will add a new layer of complexity and professionalism, making this series perfect for developers who want to understand not just *what* to build, but *how* to build it right.


## 🚗 The Project: Object Detection with a Twist

Our journey begins with a seemingly simple task: **detecting cars in images using YOLO**. But here's the twist—we're not just throwing together a quick script. Instead, we're building a foundation that will support:

- **Multiple detection models** (YOLO, TensorRT, ONNX)
- **Real-time video processing** and RTSP streams
- **Message queuing** for distributed processing
- **Comprehensive testing** and CI/CD pipelines
- **Containerization** for easy deployment
- **AND MOREEEE**

The key is that we're designing for this future from day one, not retrofitting it later.

# Building Clean Architecture with Python: A YOLO Object Detection Case Study

*Part 1: Why Hexagonal Architecture Matters from Day One*
![Hexagonal Architecture](/assets/img/hexagonal.png)
## 🏗️ Why Hexagonal Architecture? Why Now?

### The Problem with "Just Make It Work"

We've all been there. You start a project with the best intentions:

```python
# The "quick and dirty" approach
import cv2
from ultralytics import YOLO

def detect_cars(image_path):
    model = YOLO('yolov8n.pt')
    results = model(image_path)
    cars = [box for box in results[0].boxes if box.cls == 2]
    return len(cars)

# Works great! Until...
```

This approach works perfectly—until you need to:
- Switch to a different model
- Add unit tests
- Process video streams
- Deploy to production
- Handle different image formats

Suddenly, your "simple" script becomes a tangled mess of dependencies and hardcoded assumptions.

### Enter Hexagonal Architecture

**Hexagonal Architecture** (also known as Ports and Adapters) solves this problem by creating clear boundaries between your business logic and external concerns. Think of it as building a house with a solid foundation—you can change the wallpaper, add rooms, or even move the entire structure without affecting the core structure.

#### The Core Principles

1. **Business Logic at the Center**: Your core domain logic lives in the center, independent of external frameworks
2. **Ports Define Contracts**: Interfaces (ports) define what your application needs, not how it gets it
3. **Adapters Handle Implementation**: External concerns (databases, APIs, frameworks) are handled by adapters
4. **Dependency Inversion**: High-level modules don't depend on low-level modules—both depend on abstractions

### Our Project Structure

Let's see how this translates to our object detection project:

```
src/
├── domain/           # 🧠 Business logic (the "what")
│   └── filter_car_detections.py
├── ports/            # 🔌 Interfaces (the "contracts")
│   └── object_detection_port.py
├── adapters/         # 🔌 External implementations (the "how")
│   └── yolo_detection.py
├── infra/            # 🏗️ Infrastructure layer
│   └── car_count_repository.py
└── main.py          # 🚀 Application entry point
```

#### The Domain Layer: Pure Business Logic

```python
class FilterCarDetections:
    def __init__(self, class_id: int):
        self.class_id = class_id

    def filter_car_detections(self, detections: List[tuple]) -> List[tuple]:
        """Filter detections to only include cars."""
        return [detection for detection in detections 
                if detection[0][-1] == self.class_id]
```

This class knows nothing about YOLO, images, or databases. It's pure business logic that can be tested in isolation.

#### The Port: Defining the Contract

```python
class ObjectDetectionInterface(ABC):
    @abstractmethod
    def detect(self, image_path: str) -> List[Any]:
        pass
```

This interface defines what we need from any detection system, not how it works.

#### The Adapter: Implementing the Contract

```python
class YOLODetection(ObjectDetectionInterface):
    def __init__(self, model_path: str):
        self.model = YOLO(model_path)
    
    def detect(self, image_path: str) -> List[Tuple[float, ...]]:
        # YOLO-specific implementation
        detections = self.model.predict(image_path)
        return self.format_to_list_output(detections)
```
Here
This adapter handles all the YOLO-specific details while implementing our interface.

#### The Infrastructure Layer: Handling Data Persistence

```python
class CarCountRepository(RepositoryInterface):
    def __init__(self):
        self.car_count_per_image = {}
    
    def save_car_count_per_image(self, image_path: str, car_count: int):
        self.car_count_per_image[image_path] = car_count
    
    def get_car_count_per_image(self, image_path: str) -> int:
        return self.car_count_per_image.get(image_path, 0)
```

The infrastructure layer handles external concerns like data storage. In our current implementation, we're using a simple in-memory dictionary, but this could easily be swapped for a database, file system, or cloud storage without changing any business logic.

## 🎯 Why Choose Good Architecture Early?

### The Cost of Technical Debt

I've seen too many projects start with "we'll refactor later" and end up with:

- **Tightly coupled code** that's impossible to test
- **Hardcoded dependencies** that make changes risky
- **No clear boundaries** between different concerns
- **Legacy code** that nobody wants to touch

The truth is: **refactoring later is exponentially more expensive than doing it right from the start**.

### The Benefits of Starting Right

#### 1. **Testability from Day One**
--- Comming up next ---

With hexagonal architecture, we can test our business logic without any external dependencies.

#### 2. **Easy to Swap Implementations**
Want to try TensorRT instead of YOLO? Just create a new adapter:

--- Comming up next ---

No changes to your business logic required!

#### 3. **Clear Separation of Concerns**
- **Domain**: "What are we trying to achieve?"
- **Ports**: "What do we need from the outside world?"
- **Adapters**: "How do we get what we need?"

#### 4. **Future-Proof Design**
When requirements change (and they always do), you can adapt without rewriting everything.

## 🚀 What's Coming Next?

This is just the beginning! In the upcoming posts, we'll explore:

### **Phase 1: Foundation** 🏗️
- **Part 2**: Unit Testing & Test-Driven Development
- **Part 3**: GitHub Actions CI/CD Pipeline, Code Formatting & Linting

### **Phase 2: Coming Soon** 🚀
*Stay tuned for exciting new features and production-ready enhancements!*

Each post will build upon the previous one, showing you how to evolve your architecture as your requirements grow. We'll cover everything from basic testing to advanced topics like message queuing and containerization.

## 🎓 Key Takeaways

1. **Architecture matters from day one**—it's not something you add later
2. **Hexagonal architecture** provides clear boundaries and testability
3. **Dependency inversion** makes your code flexible and maintainable
4. **Good design** pays dividends as your project grows

## 🤝 Join the Journey

This series is designed for developers who want to:
- **Learn clean architecture** through practical examples
- **Understand** why design patterns matter
- **Build** production-ready software from the ground up
- **Evolve** their projects systematically

Whether you're a junior developer looking to level up or a senior developer exploring new patterns, there's something here for everyone.

## 📚 Resources and Next Steps

- **Repository up to this point**: [object_detection_repo](https://github.com/Guiandreis/car_detection_repo/tree/85e34485b1d8529184a4881078e7dc39d18800db)
- **Hexagonal Architecture**: [Alistair Cockburn's original paper](https://alistair.cockburn.us/hexagonal-architecture/)
- **Clean Architecture**: [Robert Martin's book](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
---

**Ready to dive in?** In the next post, we'll add comprehensive unit testing to our project and explore how hexagonal architecture makes testing not just possible, but actually enjoyable.

*What questions do you have about hexagonal architecture? What would you like to see covered in future posts? Let me know in the comments!*

---

*This is Part 1 of a multi-part series on building production-ready Python applications. Stay tuned for more!*
