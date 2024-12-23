# Optimizing Rectangle Detection: Performance and Memory Improvements

A while ago, I developed a program to detect rectangles from a set of points on a two-dimensional surface. Recently, I worked on improving its performance and memory usage.

## 🔧 What I Improved:
- Replaced `OrderBy` operations with `Array.Sort` for better memory efficiency.
- Added caching to avoid unnecessary computations.
- Used `CollectionsMarshal.AsSpan` for faster data access.

## 📈 Results:
- **55%** reduction in total memory allocations.
- **26%** decrease in peak live objects.
- **75%** less memory usage in the `GetOrderedPoints` method.
- **8.5%** faster execution time.

## 🛠️ Test Environment:
- **Visual Studio Version:** 17.12.0
- **.NET Version:** .NET 9
- **Build Configuration:** Release mode

These changes demonstrate how optimizing algorithms and data handling can significantly improve performance, especially in memory-intensive applications.

---

### 🔗 Explore the Project:
- [Project Link 1](https://github.com/jahanalem/RectanglesCalculator/tree/performance-improvements)  
- [Project Link 2](https://github.com/jahanalem/RectanglesCalculator/tree/main)

If you have any suggestions to further improve the performance, I’d love to hear them! 😊


![1](https://github.com/user-attachments/assets/2853e170-c1dc-4800-837c-d16f2515b7b8)

![2](https://github.com/user-attachments/assets/0cc37a9e-4d55-4b9a-aca1-e7029a24a6e0)

![3](https://github.com/user-attachments/assets/4c872a0a-9af1-4d31-923a-4a64a0f53eee)

![4](https://github.com/user-attachments/assets/895c5b19-0caa-446d-b35d-d4b075a2ff4b)

![5](https://github.com/user-attachments/assets/21566796-a69f-4579-97e4-017a5555cdda)

![6](https://github.com/user-attachments/assets/7c028cf5-6ec8-42fe-8e8d-3050aa06ed54)

![7](https://github.com/user-attachments/assets/049a4990-f02c-43bb-b95b-bcc6e254c04e)








