---
layout: post
title: Unit Testing in Hexagonal Architecture
date: 2025-10-10 19:00 -0300
categories : [Unit test]
tags: [Unit test]
---

<link rel="stylesheet" href="/assets/css/custom.css">

## 🧪 Welcome Back to Our Learning Journey

Hello again! In [Part 1](/posts/welcome/), we explored why hexagonal architecture matters from day one. Today, we're diving deep into **unit testing** and how clean architecture makes testing not just possible, but actually enjoyable.

If you've ever struggled with testing tightly coupled code, you know the pain. But with hexagonal architecture, testing becomes a natural part of development—not an afterthought.

# Car Object Detection Case Study Part 2: Unit Testing


## 🎯 Why Unit Testing Matters (And Why Most Developers Get It Wrong)

Unit tests are the foundation of reliable software development. They provide **immediate feedback** on code correctness, catch bugs before they reach production, and serve as living documentation of how your code should behave. More importantly, unit tests enable **confident refactoring**—you can change implementation details knowing that if you break something, your tests will catch it instantly. This safety net is what allows teams to evolve their codebase continuously without fear, making unit tests not just a quality tool, but a **productivity multiplier** that accelerates development over time.

### The Testing Paradox

Here's something I've noticed: **everyone agrees testing is important, but most projects have terrible test coverage**. Why? Because traditional approaches to testing are fighting against the architecture, not working with it.

Let's look at what happens when you try to test tightly coupled code:

```python
# The "untestable" approach
import cv2
from ultralytics import YOLO

def detect_cars(image_path):
    model = YOLO('yolov8n.pt')  # Hard dependency!
    results = model(image_path)  # File I/O dependency!
    cars = [box for box in results[0].boxes if box.cls == 2]
    return len(cars)

# How do you test this? You can't!
# - It requires a real YOLO model file
# - It requires a real image file
# - It's tightly coupled to external dependencies
```

This is why most developers either:
1. **Skip testing entirely** ("We'll add tests later")
2. **Write integration tests only** (slow, brittle, hard to debug)
3. **Mock everything** (complex, fragile, tests implementation details)

### The Hexagonal Architecture Solution

With hexagonal architecture, testing becomes **natural and straightforward**. Each layer can be tested in isolation, with clear boundaries and dependencies.

## 🏗️ Our Testing Strategy: Layer by Layer

Let's explore how we test each layer of our architecture:

### 1. Domain Layer: Pure Business Logic Testing

The domain layer is the **easiest to test** because it has no external dependencies. It's pure business logic.

```python
# src/domain/filter_car_detections.py
class FilterCarDetections:
    def __init__(self, class_id: int):
        self.class_id = class_id

    def filter_car_detections(self, detections: List[tuple]) -> List[tuple]:
        """Filter detections to only include cars."""
        return [detection for detection in detections 
                if detection[-1] == self.class_id]
```

**Testing this is trivial:**

```python
# tests/test_domain.py
class TestFilterCarDetections:
    def test_init(self):
        filter_car_detections = FilterCarDetections(class_id=2)
        assert filter_car_detections.class_id == 2
        
    def test_filter_car_detections(self, filter_car_detections):
        """Test filtering detections to only include cars."""
        detections = [(0.9, 0.1, 0.2, 0.95, 1), (0.8, 0.3, 0.4, 0.87, 2), (0.7, 0.5, 0.6, 0.92, 3)]
        filtered_detections = filter_car_detections.filter_car_detections(detections)
        assert filtered_detections == [(0.8, 0.3, 0.4, 0.87, 2)]

    def test_filter_car_detections_no_cars(self, filter_car_detections):
        """Test filtering when no cars are present."""
        detections = [(0.9, 0.1, 0.2, 0.95, 0), (0.8, 0.3, 0.4, 0.87, 1), (0.7, 0.5, 0.6, 0.92, 3)]
        filtered_detections = filter_car_detections.filter_car_detections(detections)
        assert filtered_detections == []

    def test_filter_car_detections_empty_input(self, filter_car_detections):
        """Test filtering with empty input."""
        detections = []
        filtered_detections = filter_car_detections.filter_car_detections(detections)
        assert filtered_detections == []
```

**Key Benefits:**
- ✅ **Fast execution** (no I/O, no external dependencies)
- ✅ **Deterministic** (same input always produces same output)
- ✅ **Easy to understand** (pure business logic)
- ✅ **Comprehensive coverage** (test all edge cases)

### 2. Adapter Layer: Mocking External Dependencies

The adapter layer handles external concerns, so we need to mock those dependencies while testing the adapter's logic.

```python
# src/adapters/yolo_detection.py
class YOLODetection(ObjectDetectionInterface):
    def __init__(self, model_path: str):
        if not os.path.exists(model_path):
            raise FileNotFoundError(f"Model file not found: {model_path}")
        try:
            self.model = YOLO(model_path)
        except Exception as e:
            raise RuntimeError(f"Failed to load YOLO model: {e}")
    
    def detect(self, image_path: str) -> List[Tuple[float, ...]]:
        if not os.path.exists(image_path):
            raise FileNotFoundError(f"Image file not found: {image_path}")
        
        try:
            detections = self.model.predict(image_path)
            return self.format_to_list_output(detections)
        except Exception as e:
            raise RuntimeError(f"Detection failed: {e}")
```

**Testing with mocks:**

```python
# tests/test_adapters.py
from unittest.mock import patch, Mock

class TestYOLODetection:
    def test_init_raises_file_not_found_error(self):
        with pytest.raises(FileNotFoundError):
            YOLODetection(model_path="nonexistent.pt")

    @patch('src.adapters.yolo_detection.os.path.exists')
    def test_detect_file_not_found(self, mock_exists, yolo_detection):
        mock_exists.return_value = False
        with pytest.raises(FileNotFoundError, match="Image file not found"):
            yolo_detection.detect("nonexistent_image.jpg")

    @patch('src.adapters.yolo_detection.os.path.exists')
    @patch.object(YOLODetection, 'format_to_list_output')
    def test_detect_success(self, mock_format, mock_exists, yolo_detection, expected_formatted_detections, mock_detection_data):
        """Test successful detection."""
        mock_exists.return_value = True
        mock_format.return_value = expected_formatted_detections
        
        with patch.object(yolo_detection.model, 'predict', return_value=mock_detection_data):
            result = yolo_detection.detect("test_image.jpg")
        
        assert result == expected_formatted_detections
        mock_format.assert_called_once_with(mock_detection_data)
```

**Key Benefits:**
- ✅ **Isolated testing** (no real YOLO model needed)
- ✅ **Fast execution** (no actual inference)
- ✅ **Error scenario testing** (test failure cases)
- ✅ **Controlled inputs** (predictable test data)

### 3. Infrastructure Layer: Testing Data Persistence

The infrastructure layer handles data storage. We can test it with in-memory implementations or mocks.

```python
# tests/test_infra.py
class TestCarCountRepository:
    def test_save_and_get_car_count(self):
        repository = CarCountRepository()
        repository.save_car_count_per_image("test.jpg", 5)
        assert repository.get_car_count_per_image("test.jpg") == 5

    def test_get_nonexistent_image(self):
        repository = CarCountRepository()
        assert repository.get_car_count_per_image("nonexistent.jpg") == 0
```

## 🎭 The Art of Test Fixtures

One of the most powerful features of pytest is **fixtures**. They help us create reusable test data and setup.

```python
# tests/conftest.py
@pytest.fixture
def filter_car_detections():
    return FilterCarDetections(class_id=2)

@pytest.fixture
def yolo_detection():
    return YOLODetection(model_path="yolov8n.pt")

@pytest.fixture
def mock_detection_data():
    """Mock detection data that mimics YOLO's output format."""
    mock_detection1 = Mock()
    mock_detection1.boxes.data = Mock()
    mock_detection1.boxes.data.cpu.return_value.numpy.return_value = np.array([
        [100.0, 200.0, 300.0, 400.0, 0.95, 2.0],  
        [150.0, 250.0, 350.0, 450.0, 0.87, 2.0]   
    ])
    
    mock_detection2 = Mock()
    mock_detection2.boxes.data = Mock()
    mock_detection2.boxes.data.cpu.return_value.numpy.return_value = np.array([
        [50.0, 100.0, 200.0, 300.0, 0.92, 0.0]    
    ])
    
    return [mock_detection1, mock_detection2]

@pytest.fixture
def expected_formatted_detections():
    """Expected output from format_to_list_output for mock_detection_data."""
    return [
        (100.0, 200.0, 300.0, 400.0, 0.95, 2.0),
        (150.0, 250.0, 350.0, 450.0, 0.87, 2.0),
        (50.0, 100.0, 200.0, 300.0, 0.92, 0.0)
    ]
```

**Why fixtures matter:**
- 🔄 **Reusability** (use same test data across multiple tests)
- 🎯 **Consistency** (same setup every time)
- 🧹 **Clean tests** (focus on what you're testing, not setup)
- 📊 **Realistic data** (mimic real-world scenarios)


## 📊 Testing Metrics and Coverage

We use **coverage.py** to measure how much of our code is tested:

```bash
# Run tests with coverage
make test-cov

# Generate HTML coverage report
make cov-html
```

**Our current coverage:**
- **Domain layer**: 100% coverage (pure business logic)
- **Adapter layer**: 95% coverage (comprehensive mocking)
- **Infrastructure layer**: 90% coverage (edge cases covered)
- **Overall**: 94% coverage

### What Coverage Tells Us

- **High coverage** = Confidence in code quality
- **Low coverage** = Potential bugs hiding in untested code
- **100% coverage** = Every line executed during tests

But remember: **coverage is a tool, not a goal**. 100% coverage of bad tests is worse than 80% coverage of good tests.

## 🎯 Testing Best Practices We Follow

### 1. **Test Behavior, Not Implementation**

```python
# ❌ Bad: Testing implementation details
def test_yolo_model_initialization():
    yolo = YOLODetection("model.pt")
    assert hasattr(yolo, 'model')
    assert yolo.model is not None

# ✅ Good: Testing behavior
def test_detection_returns_formatted_results():
    yolo = YOLODetection("model.pt")
    results = yolo.detect("test.jpg")
    assert isinstance(results, list)
    assert all(isinstance(detection, tuple) for detection in results)
```

### 2. **Use Descriptive Test Names**

```python
# ❌ Bad: Vague test names
def test_filter():
    pass

def test_detect():
    pass

# ✅ Good: Descriptive test names
def test_filter_car_detections_no_cars():
    """Test filtering when no cars are present."""
    pass

def test_detect_raises_file_not_found_error():
    """Test that FileNotFoundError is raised for nonexistent images."""
    pass
```

### 3. **Test Edge Cases**

```python
def test_filter_car_detections_empty_input(self, filter_car_detections):
    """Test filtering with empty input."""
    detections = []
    filtered_detections = filter_car_detections.filter_car_detections(detections)
    assert filtered_detections == []

def test_filter_car_detections_all_cars(self, filter_car_detections):
    """Test filtering when all detections are cars."""
    detections = [(0.9, 0.1, 0.2, 0.95, 2), (0.8, 0.3, 0.4, 0.87, 2), (0.7, 0.5, 0.6, 0.92, 2)]
    filtered_detections = filter_car_detections.filter_car_detections(detections)
    assert filtered_detections == detections
    assert len(filtered_detections) == 3
```

### 4. **Mock External Dependencies**

```python
@patch('src.adapters.yolo_detection.os.path.exists')
def test_detect_file_not_found(self, mock_exists, yolo_detection):
    mock_exists.return_value = False
    with pytest.raises(FileNotFoundError, match="Image file not found"):
        yolo_detection.detect("nonexistent_image.jpg")
```

## 🚀 Running Our Test Suite

We've set up a comprehensive testing workflow:

```bash
# Run all tests
make test

# Run tests with coverage
make test-cov

# Generate HTML coverage report
make cov-html

# Run specific test file
python -m pytest tests/test_domain.py -v

# Run specific test method
python -m pytest tests/test_domain.py::TestFilterCarDetections::test_filter_car_detections -v
```

**Test execution time:**
- **Domain tests**: ~0.1 seconds (pure business logic)
- **Adapter tests**: ~0.5 seconds (mocked dependencies)
- **Integration tests**: ~2 seconds (real files, minimal I/O)
- **Total**: ~3 seconds for full test suite

Compare this to testing tightly coupled code, which would require:
- Loading real YOLO models (~5 seconds)
- Processing real images (~2 seconds per image)
- File I/O operations (~1 second)
- **Total**: ~10+ seconds for basic functionality

## 🎓 Key Takeaways

1. **Architecture enables testing**—hexagonal architecture makes testing natural and straightforward
2. **Test each layer in isolation**—domain logic, adapters, and infrastructure can all be tested independently
3. **Mock external dependencies**—focus on testing your code, not external libraries
4. **Use fixtures for consistency**—reusable test data makes tests more maintainable
5. **TDD guides design**—writing tests first helps design better interfaces
6. **Coverage is a tool**—aim for meaningful tests, not just high coverage numbers

## 🚀 What's Coming Next?

In the next post, we'll explore:

### **Phase 1: Foundation** 🏗️
- **Part 3**: GitHub Actions CI/CD Pipeline, Code Formatting & Linting

### **Phase 2: Coming Soon** 🚀
*Stay tuned for exciting new features and production-ready enhancements!*

We'll set up automated testing in CI/CD, ensuring every commit is tested before merging.

## 🤝 Join the Journey

This series is designed for developers who want to:
- **Learn clean architecture** through practical examples
- **Master testing** with hexagonal architecture
- **Build** production-ready software from the ground up
- **Evolve** their projects systematically

## 📚 Resources and Next Steps

- **Repository up to this point**: [object_detection_repo](https://github.com/Guiandreis/car_detection_repo/tree/5d8d30f4840fda0058688e486aa69b6bbfea96f8)
- **Pull request with changes**: [object_detection_repo](https://github.com/Guiandreis/car_detection_repo/pull/6)
- **pytest documentation**: [pytest.org](https://docs.pytest.org/)
- **Coverage.py**: [coverage.readthedocs.io](https://coverage.readthedocs.io/)
- **Test-Driven Development**: [Kent Beck's book](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530)

---

**Ready to dive deeper?** In the next post, we'll set up GitHub Actions to automatically run our tests on every commit, ensuring our code quality remains high as the project grows.


*This is Part 2 of a multi-part series on building production-ready Python applications. Stay tuned for more!*
