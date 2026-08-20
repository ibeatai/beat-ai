## 8.9用PyTorch实现多分类神经网络

上一节，我们自己实现了一个多分类神经网络来对手写数字进行识别，它的目的是让你了解神经网络底层的原理，这一节，我们就利用PyTorch来实现同样的功能，你会发现我们的工作会变简单不少。

### 8.9.1定义模型

```python
class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(28 * 28, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Linear(64, 10)
        )

    def forward(self, x):
        return self.model(x)
```

可以看到定义了一个神经网络的类，它继承自`nn.Module`，我们就可以直接利用PyTorch定义好的`Linear`和`ReLU`。在初始化函数里，我们定义了一个`nn.Sequential`的对象，它会顺序链接内部各个模块。`forward`方法就是直接调用我们定义的model。

### 8.9.2准备工作

```python
# 参数设置
batch_size = 64
learning_rate = 0.1
num_epochs = 10
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# 数据加载
train_dataset = MNISTDataset(r'./data/mnist/mnist_train.csv/mnist_train.csv')
train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
test_dataset = MNISTDataset(r"./data/mnist/mnist_test.csv/mnist_test.csv")
test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)

# 模型、损失函数、优化器
model = NeuralNetwork().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=learning_rate)
```

我们利用PyTorch内部定义的交叉熵损失函数，以及优化器。

### 8.9.3训练模型

```python
# 训练过程
model.train()
for epoch in range(num_epochs):
    total_loss = 0
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)

        outputs = model(images)
        loss = criterion(outputs, labels) # 计算loss

        optimizer.zero_grad() # 清理梯度
        loss.backward() # 反向传播
        optimizer.step() # 更新参数

        total_loss += loss.item()

    avg_loss = total_loss / len(train_loader)
    print(f"Epoch {epoch+1}/{num_epochs}, Loss: {avg_loss:.4f}")
```

### 8.9.4测试模型

```python
model.eval()
correct = 0
total = 0
with torch.no_grad():
    for images, labels in test_loader:
        images, labels = images.to(device), labels.to(device)
        outputs = model(images)
        preds = torch.argmax(outputs, dim=1)
        correct += (preds == labels).sum().item()
        total += labels.size(0)

print(f"Test Accuracy: {100 * correct / total:.2f}%")
```

### 8.9.5完整代码

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, Dataset

# 自定义数据集
class MNISTDataset(Dataset):
    def __init__(self, file_path):
        self.images, self.labels = self._read_file(file_path)

    def _read_file(self, file_path):
        images = []
        labels = []
        with open(file_path, 'r') as f:
            next(f)  # 跳过标题行
            for line in f:
                items = line.strip().split(",")
                images.append([float(x) for x in items[1:]])
                labels.append(int(items[0]))
        return images, labels

    def __getitem__(self, index):
        image = torch.tensor(self.images[index], dtype=torch.float32).view(-1)
        image = image / 255.0  # 归一化
        image = (image - 0.1307) / 0.3081  # 标准化
        label = torch.tensor(self.labels[index], dtype=torch.long)
        return image, label

    def __len__(self):
        return len(self.images)

# 模型定义
class NeuralNetwork(nn.Module):
    def __init__(self):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(28 * 28, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Linear(64, 10)
        )

    def forward(self, x):
        return self.model(x)

# 参数设置
batch_size = 64
learning_rate = 0.1
num_epochs = 10
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

# 数据加载
train_dataset = MNISTDataset(r'./data/mnist/mnist_train.csv/mnist_train.csv')
train_loader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
test_dataset = MNISTDataset(r"./data/mnist/mnist_test.csv/mnist_test.csv")
test_loader = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)

# 模型、损失函数、优化器
model = NeuralNetwork().to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=learning_rate)

# 训练过程
model.train()
for epoch in range(num_epochs):
    total_loss = 0
    for images, labels in train_loader:
        images, labels = images.to(device), labels.to(device)

        outputs = model(images)
        loss = criterion(outputs, labels)

        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        total_loss += loss.item()

    avg_loss = total_loss / len(train_loader)
    print(f"Epoch {epoch+1}/{num_epochs}, Loss: {avg_loss:.4f}")

# 测试过程
model.eval()
correct = 0
total = 0
with torch.no_grad():
    for images, labels in test_loader:
        images, labels = images.to(device), labels.to(device)
        outputs = model(images)
        preds = torch.argmax(outputs, dim=1)
        correct += (preds == labels).sum().item()
        total += labels.size(0)

print(f"Test Accuracy: {100 * correct / total:.2f}%")
```

运行后，你会得到和我们之前手动实现的神经网络差不多的精度。
