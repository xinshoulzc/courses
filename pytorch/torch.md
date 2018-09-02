# pytorch

## matrix

- torch.Tensor

```python
x = torch.empty(5, 3) # 5x3 matrix with empty value(enough low)
x = torch.rand(5, 3)
x = torch.zeros(5, 3, dtype=torch.long)
x = torch.tensor([5.5 3]) # construct tensor from data
y = x.new_ones(5, 3, dtype=torch.double) # create y same type with x
x = torch.randn_like(y, dtype=torch.float) # override dtype; create x same size with y

# resize tensor
x = torch.randn(4, 4)
y = x.view(16)
z = x.view(-1, 8) # -1 same with tensorflow

# tensor to numpy
a = torch.ones(5)
b = a.numpy() 
# numpy to tensor
a = np.ones(5)
b = torch.from_numpy(a)
```

## Neural network

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim

class Net(nn.Module):

    def __init__(self):
        super(Net, self).__init__()
        # class torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)
        # padding attention
        self.conv1 = nn.Conv2d(1, 6, 5)
        self.conv2 = nn.Conv2d(6, 16, 5)

        # y = w * x + b nn.Linear(in_feature, out_feature)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = F.max_pool2d(F.relu(self.conv1(x)), (2, 2))
        x = F.max_pool2d(F.relu(self.conv1(x)), 2) # 2 same with (2, 2)
        x = x.view(-1, self.num_flat_feature(x))
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

    def num_flat_feature(self, x):
        size = x.size()[1:]
        num_feature = 1
        for s in size:
            num_feature *= s
        return num_feature

net = Net()
params = net.parameters()

# Loss function
output = net(input)
target = torch.randn(10)
target = target.view(1, -1)
citerion = nn.MSELoss()

loss = criterion(output, target)

net.zero_grad()

print("conv1.bias.grad before backward")
print(net.conv1.bias.grad)

loss.backward()

print("conv1.bias.grad after backward")
print(net.conv1.bias.grad)

optimizer = optim.SGD(net.parameters(), lr=0.01)

optimizer.zero_grad()
output = net(input)
loss = criterion(output, target)
loss.backward()
optimizer.step() # Does the update
```

## NLP pytorch
```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim

torch.manual_seed(1)
F.softmax(data, dim=0)
lstm = nn.LSTM(3, 3) # input dims are 3, output dims are 3

# input data
inputs = [torch.randn(1, 3) for _ in range(5)]

# init hidden state
hidden= (torch.randn(1, 1, 3), torch.randn(1, 1, 3))

for i in inputs:
    out, hidden = lstm(i.view(1, 1, -1), hidden)

inputs = torch.cat(inputs).view(len(inputs), 1, -1)
hidden = (torch.randn(1, 1, 3), torch.randn(1, 1, 3))  # clean out hidden state
out, hidden = lstm(inputs, hidden)
print(out)
print(hidden)

```






