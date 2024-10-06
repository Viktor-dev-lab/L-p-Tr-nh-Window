## Phần Giao Diện

```xaml
<Window x:Class="ModelMVVM.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ModelMVVM"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">

    <Window.DataContext>
        <local:personModel/>
    </Window.DataContext>
    
    <Grid>
        <StackPanel Width="300">
            <TextBox Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}" Width="200" />
            <Label Content="{Binding Name}" />
            <TextBox/>
        </StackPanel>
    </Grid>
</Window>
```

### Bước 1 Khi `MainWindow` khởi động:
- Trong XAML, khi bạn thiết lập `DataContext` cho `MainWindow` là một đối tượng của `personModel`, thì một thể hiện của `personModel` được tạo ra.

```xml
<Window.DataContext>
    <local:personModel/>
</Window.DataContext>
```

### Mã Lớp `personModel`:

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ModelMVVM
{
   internal class personModel : ViewModelNoti
   {
      private string _name;

      public string Name
      {
         get { return _name; }
         set 
         {
            _name = value;
            OnPropertyChanged();
         }
      }

      public personModel()
      {
         Task.Run(() =>
         {
            while(true)
            {
               Random r = new Random();
               Name = r.Next(1,999).ToString();
               Debug.WriteLine($"Name: {Name}");
               Thread.Sleep(500);
            }
         });
      }
   }
}
```

### Mã Lớp `ViewModelNoti`:

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Text;
using System.Threading.Tasks;

namespace ModelMVVM
{
   public abstract class ViewModelNoti : INotifyPropertyChanged
   {
      // Sự kiện PropertyChanged từ INotifyPropertyChanged
      public event PropertyChangedEventHandler PropertyChanged;

      // Phương thức OnPropertyChanged thông báo khi một thuộc tính thay đổi
      // CallerMemberName tự động lấy tên của thuộc tính gọi phương thức này, không cần phải truyền thủ công
      public void OnPropertyChanged([CallerMemberName] string propertyName = null)
      {
         PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
      }
   }
}
```

### Bước 2 Constructor của `personModel`:
- Ngay khi một thể hiện của `personModel` được tạo, constructor của lớp này sẽ được gọi.
- Trong constructor, bạn có một vòng lặp bất đồng bộ (`Task.Run`) mà tạo ra các giá trị ngẫu nhiên cho thuộc tính `Name` và gọi `OnPropertyChanged()`.
- Lúc này, giá trị `_name` ban đầu là rỗng (`null` hoặc `""`), nhưng khi vòng lặp thực thi lần đầu tiên, nó sẽ gán một giá trị ngẫu nhiên cho `Name` (ví dụ: `Random r = new Random(); Name = r.Next(1,999).ToString();`).

### Bước 3 Gọi `OnPropertyChanged()`:
- Khi bạn gán một giá trị cho `Name`, setter của thuộc tính này được gọi. Trong setter:
```csharp
set 
{
    _name = value;          // Cập nhật giá trị
    OnPropertyChanged();     // Thông báo cho UI
}
```
- `OnPropertyChanged()` sẽ được gọi, và sự kiện `PropertyChanged` sẽ được kích hoạt. Điều này sẽ thông báo cho UI rằng giá trị của `Name` đã thay đổi.

### Bước 4 Cập nhật UI:
- Khi sự kiện `PropertyChanged` được kích hoạt, UI sẽ nhận được thông báo và tự động cập nhật các control liên kết với thuộc tính `Name`.
- Cả `TextBox` và `Label` sẽ hiển thị giá trị mới của `Name`.
