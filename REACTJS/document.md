# Hooks trong React

## 1. React Hooks là gì?

Hook trong react là các hàm đặc biệt cho phép sử dụng trạng thái (state) và các tính năng vòng đời (lifecycle) của React trong các hàm component chức năng (functional components) thay vì chỉ trong các class component. Hook giúp viết code ngắn gọn, dễ đọc, dễ tái sử dụng logic trạng thái giữa các component khác nhau và giảm thiểu sự phức tạp trong quá trình quản lý ứng dụng.

### 1.1. Những điểm chính về Hook:

- Sử dụng State trong Functional Components:  
  Trước Hooks, bạn cần dùng class component để quản lý state. Giờ đây, với Hooks như useState, bạn có thể khai báo và cập nhật state ngay trong thành phần hàm.
- Tái sử dụng Logic:  
  Bạn có thể tạo ra Hook tùy chỉnh (custom Hook) để trích xuất logic có trạng thái và chia sẻ nó một cách dễ dàng giữa nhiều component khác nhau, thay vì sử dụng các kỹ thuật phức tạp như render props hay higher-order components.
- Truy cập Tính năng Vòng đời:  
  Hooks như useEffect cho phép bạn thực hiện các hành động phụ thuộc vào vòng đời của component, tương tự như các phương thức trong componentDidMount hoặc componentDidUpdate của class components.
- Quy tắc đặt tên:  
  Tên của một Hook luôn phải bắt đầu bằng tiền tố use-, ví dụ: useState, useEffect, useContext, useRef.
- Tuân thủ thứ tự gọi:  
  Hooks phải được gọi ở cấp cao nhất của thành phần hàm, không được gọi bên trong vòng lặp, điều kiện hoặc các hàm lồng nhau khác, để React có thể quản lý trạng thái chính xác.

## 2. Tại sao chúng ta cần sử dụng Hooks?

Quản lý state và life cycle của component dễ dàng hơn: Trước khi có React Hooks, state và life cycle của React components thường chỉ được quản lý trong Class Components. Functional Components không có khả năng quản lý trạng thái hoặc thực hiện các tác vụ sau khi render. React Hooks giúp bạn sử dụng trạng thái và vòng đời trong Functional Components một cách dễ dàng và hiệu quả. Dưới đây là cách quản lý state và life cycle của React components trong Class Component:

```javascript
import React, { Component } from 'react';

class ExampleComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }

  componentDidMount() {
    // Thực hiện sau khi component đã được render lần đầu
    console.log('Component did mount');
  }

  componentDidUpdate(prevProps, prevState) {
    // Thực hiện sau mỗi lần component được cập nhật
    console.log('Component did update');
  }

  componentWillUnmount() {
    // Thực hiện trước khi component bị unmount
    console.log('Component will unmount');
  }

  incrementCount = () => {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.incrementCount}>Increment</button>
      </div>
    );
  }
}
```

Còn đây là cách quản lý trong Functional Component khi sử dụng Hooks:

```javascript
import React, { useState, useEffect } from 'react';

function ExampleComponent() {
  // Khởi tạo state bằng useState
  const [count, setCount] = useState(0);

  // Sử dụng useEffect thay thế cho lifecycle methods
  useEffect(() => {
    // Thực hiện sau khi component đã được render lần đầu (tương đương với componentDidMount)
    console.log('Component did mount');

    // Cleanup function tương đương với componentWillUnmount
    return () => {
      console.log('Component will unmount');
    };
  }, []); // [] đại diện cho dependencies, rỗng nghĩa là sẽ chỉ thực hiện một lần sau lần render đầu tiên

  // Sử dụng cú pháp arrow function để cập nhật state
  const incrementCount = () => {
    setCount(count + 1);
  }

  // Render component
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCount}>Increment</button>
    </div>
  );
}

export default ExampleComponent;
```

Trông gọn gàng và dễ hiểu hơn nhiều đúng không?

Giảm sự phức tạp của class components: Class components có thể trở nên phức tạp khi bạn cần quản lý nhiều vòng đời và trạng thái khác nhau. React Hooks giảm đi sự phức tạp này bằng cách cho phép bạn sử dụng các hook riêng lẻ để quản lý từng khía cạnh của component, làm cho mã nguồn trở nên gọn gàng hơn.

Cải thiện hiệu suất và tối ưu hóa việc render: React Hooks giúp bạn tối ưu hóa hiệu suất của ứng dụng bằng cách cho phép bạn tối ưu hóa việc render components thông qua useMemo, useCallback, và các hooks khác. Điều này giúp tránh việc render không cần thiết và cải thiện hiệu suất tổng thể.

Tái sử dụng logic và trạng thái: React Hooks cho phép bạn tái sử dụng logic và trạng thái dễ dàng hơn. Bạn có thể viết các custom hooks để chia sẻ logic giữa các components khác nhau, giúp giảm lặp lại mã nguồn và làm cho mã nguồn trở nên linh hoạt hơn.

## 3. Một số Hook thường dùng trong React

Sau khi đã hiểu lý do tại sao nên dùng Hooks, chúng ta cùng tìm hiểu xem những Hooks nào thường được sử dụng trong thực tế nhé. React đã cung cấp sẵn cho chúng ta rất nhiều Hooks như useState(), useEffect(), useCallback(),... Ngoài những hooks trên các bạn hoàn toàn có thể tạo ra hooks cho riêng mình để sử dụng vào những trường hợp khác nhau.

### 3.1 useState()

#### 3.1.1 useState() là gì?

useState() là một hook trong React được sử dụng để khởi tạo và quản lý trạng thái (state) trong Functional Components. Hook này cho phép thêm trạng thái vào Functional Components mà trước đây chỉ có thể được quản lý trong Class Components. useState() trả về một mảng với hai phần tử:

- State variable (biến trạng thái): Đây là biến mà bạn sử dụng để lưu trữ giá trị trạng thái của component.
- State updater function (hàm cập nhật trạng thái): Đây là hàm bạn sử dụng để cập nhật giá trị trạng thái. Khi bạn gọi hàm này với một giá trị mới, nó sẽ cập nhật giá trị trạng thái và render lại component để cập nhật giao diện người dùng.

Dưới đây là một ví dụ đơn giản về cách sử dụng useState():

```javascript
import React, { useState } from 'react';

function ExampleComponent() {
  // Khởi tạo trạng thái với giá trị ban đầu là 0
  const [count, setCount] = useState(0);

  // Hàm tăng giá trị trạng thái
  const incrementCount = () => {
    setCount(count + 1); // Gọi hàm setCount để cập nhật trạng thái
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCount}>Increment</button>
    </div>
  );
}
```

Trong ví dụ này, chúng ta sử dụng useState(0) để khởi tạo trạng thái count với giá trị ban đầu là 0. Khi người dùng nhấn nút "Increment," hàm incrementCount được gọi, và chúng ta sử dụng setCount để cập nhật giá trị trạng thái. React sẽ tự động render lại component để cập nhật giao diện người dùng.

#### 3.1.2 Một số lưu ý khi sử dụng useState()

- Bắt buộc phải khởi tạo giá trị ban đầu cho useState(), điều này giúp đảm bảo rằng trạng thái không bao giờ là undefined. useState() có thể lưu trữ các giá trị dạng strings, numbers, booleans, arrays, objects. Ví dụ: const [count, setCount] = useState(0);.
- Giá trị của trạng thái sẽ quay về giá trị khởi tạo sau mỗi lần refresh trang
- Component sẽ render lại mỗi khi giá trị của trạng thái bị thay đổi
- useState() không ghi đè các trạng thái mà thay thế trạng thái cũ bằng giá trị của trạng thái mới
- Không bao giờ thay đổi trạng thái trực tiếp bằng cách gán giá trị mới, ví dụ: count = count + 1. Hãy sử dụng setCount để cập nhật lại trạng thái.
- Hàm cập nhật trạng thái có thể nhận tham số là giá trị mới hoặc một hàm trả về giá trị mới dựa trên trạng thái trước đó. Ví dụ: setCount(count + 1) hoặc setCount(prevCount => prevCount + 1).
- Chỉ sử dụng useState cho những trạng thái cần thiết và không tạo ra quá nhiều trạng thái không cần thiết trong component của bạn.

#### 3.1.3 Khi nào thì nên sử dụng useState()

useState trong React nên được sử dụng khi bạn cần quản lý và theo dõi trạng thái của functional components. Dưới đây là một số tình huống nên sử dụng useState:

- Quản lý trạng thái của giao diện người dùng: Khi bạn muốn lưu trữ và cập nhật thông tin trạng thái liên quan đến giao diện người dùng, như số lượng sản phẩm trong giỏ hàng, giá trị của các trường input, hoặc trạng thái hiển thị/hide của các phần tử.
- Lưu trữ dữ liệu tạm thời: Khi bạn cần lưu trữ các dữ liệu tạm thời mà không cần sử dụng Redux hoặc Context API, useState là một cách tiện lợi để quản lý trạng thái tạm thời này.
- Xử lý trạng thái của form: useState thường được sử dụng để lưu trạng thái của form như giá trị của các trường input, chọn các mục trong danh sách, và kiểm tra tính hợp lệ của form trước khi gửi dữ liệu.
- Các trạng thái cần được cập nhật sau khi component đã render: useState cho phép bạn cập nhật trạng thái sau khi component đã render thông qua sự kích hoạt của các sự kiện hoặc lệnh gọi API. Điều này giúp bạn cập nhật giao diện người dùng khi trạng thái thay đổi.
- Quản lý các hiệu ứng tương tác đơn giản: Nếu bạn cần quản lý các hiệu ứng tương tác đơn giản như sự thay đổi của nút bấm hoặc sự xuất hiện/ẩn của các phần tử, useState có thể được sử dụng để theo dõi và cập nhật trạng thái của các hiệu ứng này.

Tóm lại, useState nên được sử dụng khi bạn cần quản lý trạng thái trong functional components của React. Nó giúp bạn làm cho components trở nên tương tác và có khả năng cập nhật dễ dàng, đồng thời giúp tạo ra các ứng dụng React hiệu quả với mã nguồn dễ đọc và dễ bảo trì.

### 3.2 useEffect()

#### 3.2.1 useEffect() là gì?

useEffect là một trong những hooks quan trọng trong React, được sử dụng để thực hiện các tác vụ phụ (side effects) trong Functional Components. Các tác vụ phụ bao gồm lệnh gọi API, thay đổi trạng thái, đăng ký, lắng nghe và hủy đăng ký sự kiện (event), và các tác vụ không thuộc về việc render giao diện người dùng. useEffect giúp thực hiện các tác vụ này tại các thời điểm cụ thể trong vòng đời của component.

Dưới đây là một ví dụ đơn giản về cách sử dụng useEffect() để thực hiện việc gọi API bằng thư viện axios:

```javascript
import React, { useState, useEffect } from "react";
import axios from "axios";

function ExampleComponent() {
  const [data, setData] = useState({});

  const getDataFromAPI = async () => {
    const response = await axios.get(
      "https://jsonplaceholder.typicode.com/todos/1"
    );
    setData(response.data);
  };

  useEffect(() => {
    getDataFromAPI();
  }, []);

  return <div>{data.title}</div>;
}

export default ExampleComponent;
```

Luồng xử lý của ví dụ trên diễn ra như sau:

- Khai báo state data với giá trị khởi tạo là 1 object rỗng {}
- Tạo hàm getDataFromAPI() sử dụng thư viện axios để call api
- Gọi hàm getDataFromAPI() bên trong useEffect(). Hàm này sẽ được gọi khi component được render lần đầu tiên
- Sau khi lấy được dữ liệu thì gán lại state data với dữ liệu mới bằng setData
- Dữ liệu mới được đưa lên UI để hiển thị cho người dùng

#### 3.2.2 Dependencies trong useEffect()

Dependencies trong useEffect được sử dụng để kiểm soát cách hoạt động của useEffect() khi useEffect được gọi lại. Có 3 trường hợp sử dụng useEffect():

Không cung cấp dependencies:

```javascript
import React, { useState, useEffect } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);

  // useEffect không có dependencies
  useEffect(() => {
    console.log('useEffect ran.');
  });

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase Count</button>
    </div>
  );
}

export default MyComponent;
```

Trong ví dụ này, useEffect không có dependencies, vì vậy nó sẽ chạy sau mỗi lần component được render. Khi bạn nhấn vào nút "Increase Count" để tăng giá trị của count, useEffect sẽ luôn được gọi lại và in ra "useEffect ran." trên console. Điều này có nghĩa rằng nó chạy sau mỗi lần tương tác với component, dù giá trị count có thay đổi hay không.

Dependencies là mảng rỗng

```javascript
import React, { useState, useEffect } from 'react';

function MyComponent() {
  const [count, setCount] = useState(0);

  // useEffect với mảng dependencies rỗng
  useEffect(() => {
    console.log('useEffect ran.');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase Count</button>
    </div>
  );
}

export default MyComponent;
```

Trong ví dụ này, useEffect có một mảng dependencies rỗng, []. Điều này đồng nghĩa với việc nó chỉ chạy một lần sau khi component được render lần đầu tiên. Khi bạn nhấn vào nút "Increase Count" để tăng giá trị của count, useEffect sẽ không bao giờ được gọi lại. Điều này thích hợp cho các tác vụ mà bạn chỉ muốn thực hiện một lần và không muốn chúng phụ thuộc vào bất kỳ giá trị nào. Cách hoạt động tương tự như componentDidMount của Class Component.

Dependencies là props hoặc state

Khi bạn truyền dependencies là props hoặc state, useEffect sẽ được gọi lại mỗi khi giá trị của dependencies thay đổi. Điều này thường được sử dụng để thực hiện các tác vụ phụ thuộc vào sự thay đổi của props hoặc state. Cơ chế này tương tự như bạn sử dụng Life Cycle componentDidUpdate và shouldComponentUpdate của Class Component.

```javascript
import React, {useEffect, useState} from 'react'

function ExampleComponent() {
  const [count, setCount] = useState(0);

  // thay đổi giá trị của count
  const incrementCount = () => {
    setCount(count + 1);
  }

  // không thay đổi giá trị của count
  const doNothing = () => {
    setCount(count);
  }

  // useEffect callback được gọi khi state thay đổi so với giá trị trước đó
  useEffect(() => {
    console.log("useEffect ran.");
  }, [count])

  return (
    <section>
      <h1>{count}</h1>

      <button onClick={incrementCount}>Tăng thêm</button>
      <button onClick={doNothing}>Không có gì xảy ra</button>
    </section>
  );
}
```

Trong ví dụ này, useEffect có count là một dependency. Khi gọi đến hàm incrementCount(), giá trị của count thay đổi tăng lên 1, lúc này useEffect() được gọi lại và in dòng useEffect ran. ra console. Nhưng khi gọi đến hàm doNothing(), giá trị của state count không thay đổi nên useEffect() không được gọi lại trong trường hợp này.

#### 3.2.3 Clean up useEffect()

Khi bạn sử dụng useEffect trong React để thực hiện các tác vụ side effect, bạn có thể cần thực hiện một số tác vụ dọn dẹp sau khi component bị unmount hoặc khi dependencies thay đổi. Điều này giúp tránh rò rỉ bộ nhớ (memory leak) hoặc xảy ra lỗi khi chuyển sang component khác.

Ví dụ, khi bạn thêm một sự kiện như sự kiện cuộn trang (scroll) bằng addEventListener trong React, bạn cũng cần dọn dẹp (remove) sự kiện khi component bị unmount để tránh rò rỉ bộ nhớ.

```javascript
import React, { useEffect } from 'react';

function ScrollListenerComponent() {
  // Hàm xử lý sự kiện cuộn
  const handleScroll = () => {
    // Xử lý sự kiện cuộn ở đây
    console.log('Scrolled');
  };

  useEffect(() => {
    // Thêm sự kiện cuộn khi component được mount
    window.addEventListener('scroll', handleScroll);

    // Hàm dọn dẹp, được gọi khi component bị unmount
    return () => {
      window.removeEventListener('scroll', handleScroll);
    };
  }, []);

  return (
    <div>
      <p>Scroll down to trigger the event.</p>
    </div>
  );
}

export default ScrollListenerComponent;
```

Luồng hoạt động của ví dụ trên như sau:

- tạo một hàm handleScroll để xử lý sự kiện cuộn.
- Trong useEffect, chúng ta thêm một sự kiện cuộn bằng window.addEventListener khi component được mount.
- Chúng ta trả về một hàm clean up từ useEffect, và trong hàm này, chúng ta gỡ bỏ sự kiện cuộn bằng window.removeEventListener khi component bị unmount. Điều này đảm bảo rằng sự kiện cuộn sẽ không được theo dõi sau khi component bị unmount.

#### 3.2.4 Khi nào nên sử dụng useEffect()

- Thực hiện các tác vụ không liên quan đến việc rendering: useEffect() thường được sử dụng để thực hiện các tác vụ như gọi API, thao tác với DOM, thiết lập các lệnh lắng nghe sự kiện (event listeners), và các tác vụ khác không liên quan trực tiếp đến việc rendering của component.
- Quản lý lý thuyết hợp đồng (lifecycle): useEffect() cho phép bạn thực hiện các tác vụ tại các điểm quan trọng trong vòng đời của một component React, như sau khi component được tạo (componentDidMount) hoặc sau khi nó được cập nhật (componentDidUpdate). Điều này giúp bạn quản lý tất cả các tác vụ "side effects" một cách dễ dàng.
- Theo dõi và xử lý sự thay đổi của props hoặc state: Bạn có thể sử dụng useEffect() để theo dõi sự thay đổi của props hoặc state và thực hiện các tác vụ phản ứng dựa trên những thay đổi đó.
- Thực hiện các tác vụ liên quan đến dữ liệu cần lấy từ API hoặc dự báo: Nếu bạn cần lấy dữ liệu từ một API hoặc thực hiện các tính toán phức tạp, bạn có thể sử dụng useEffect() để thực hiện các tác vụ này.
- Lắng nghe và xử lý sự kiện gốc (native events): useEffect() cũng thích hợp để lắng nghe các sự kiện gốc của trình duyệt như sự kiện resize, scroll, hoặc bất kỳ sự kiện nào mà bạn muốn theo dõi và xử lý.
- Làm sạch tác vụ khi component unmount (componentWillUnmount): Bạn có thể sử dụng useEffect() với một hàm trả về để làm sạch các tác vụ khi component bị unmount.

### 3.3 useLayoutEffect()

#### 3.3.1 useLayoutEffect() là gì?

useLayoutEffect là một hook tương tự useEffect trong React, nhưng nó được gọi đồng bộ ngay sau khi các thay đổi trong DOM được áp dụng, trước khi trình duyệt cập nhật giao diện người dùng.

Một ví dụ thường gặp nhất khi sử dụng useLayoutEffect() là khi bạn thực hiện các tác vụ liên quan đến layout hoặc đồ họa.

```javascript
import React, { useLayoutEffect, useState } from 'react';

function DrawCanvas() {
  const [ctx, setCtx] = useState(null);

  useLayoutEffect(() => {
    const canvas = document.getElementById('myCanvas');
    const context = canvas.getContext('2d');

    // Thực hiện vẽ hình trước khi component render
    context.fillStyle = 'red';
    context.fillRect(0, 0, 100, 100);

    setCtx(context);
  }, []);

  return (
    <div>
      <canvas id="myCanvas" width="200" height="200" />
      <button onClick={() => {
        if (ctx) {
          // Vẽ hình sau khi component đã render
          ctx.fillStyle = 'blue';
          ctx.fillRect(100, 100, 100, 100);
        }
      }}>
        Draw on Canvas
      </button>
    </div>
  );
}

export default DrawCanvas;
```

Ở ví dụ trên, chúng ta sử dụng useLayoutEffect để thực hiện việc vẽ hình trước khi component render. Việc này đảm bảo rằng hình đã được vẽ và cập nhật trong DOM trước khi người dùng thấy giao diện. Sau khi component đã render, khi người dùng nhấn vào nút "Draw on Canvas," chúng ta có thể thực hiện các tác vụ vẽ khác với bộ công cụ context đã được thiết lập trước đó.

Việc sử dụng useLayoutEffect trong trường hợp này đảm bảo rằng việc vẽ hình sẽ xảy ra ngay trước khi giao diện người dùng được cập nhật, và người dùng sẽ thấy hình đã được vẽ mà không có độ trễ.

#### 3.3.2 So sánh useLayoutEffect() và useEffect()

Giống nhau:

- Cả hai đều được sử dụng để thực hiện các tác vụ phụ thuộc vào hiệu ứng phụ (side effects): Cả useLayoutEffect và useEffect đều được sử dụng để thực hiện các tác vụ không đồng bộ hoặc liên quan đến layout, đồ họa, tương tác với bên ngoài (API calls, DOM manipulation, ...) trong React.
- Cả hai đều có thể sử dụng dependencies: Cả hai hook đều cho phép bạn sử dụng dependencies (một mảng các giá trị) để kiểm soát khi hook được gọi lại. Hook sẽ chỉ được gọi lại khi một hoặc nhiều giá trị trong mảng dependencies thay đổi.
- Cả hai đều được sử dụng trong functional components: Cả useLayoutEffect và useEffect đều được sử dụng trong các functional components của React.

Khác nhau:

| useEffect | useLayoutEffect |
|-----------|-----------------|
| Thời điểm chạy | Chạy bất đồng bộ, sau khi component đã được render và giao diện người dùng đã được cập nhật. Điều này thích hợp cho hầu hết các tác vụ bất đồng bộ. | Chạy đồng bộ, ngay sau khi các thay đổi trong DOM đã được áp dụng và trước khi giao diện người dùng được cập nhật. Điều này đảm bảo rằng tác vụ được thực hiện trước khi người dùng thấy bất kỳ thay đổi gì trong giao diện. Quá trình này có thể làm chậm quá trình render và tạo cảm giác có độ trễ. |
| Hiệu suất | Thích hợp cho hầu hết các tác vụ bất đồng bộ. useEffect() thường được sử dụng khi bạn cần thực hiện các tác vụ phụ thuộc vào hiệu ứng phụ (side effects) mà không cần phải thực hiện chúng đồng bộ trước khi giao diện người dùng được cập nhật. useEffect không làm chậm quá trình render và làm cho ứng dụng hoạt động nhanh hơn. | Nếu không cẩn thận, useLayoutEffect có thể làm chậm quá trình render và gây ra cảm giác có độ trễ. useLayoutEffect thường được sử dụng cho các tác vụ liên quan đến layout hoặc đồ họa, và khi cần đảm bảo rằng tác vụ được thực hiện ngay lập tức. |

#### 3.3.3 Khi nào nên sử dụng useLayoutEffect()?

- Khi bạn cần thực hiện các thay đổi giao diện ngay sau khi rendering và trước khi trình duyệt cập nhật giao diện người dùng. useLayoutEffect() sẽ đảm bảo rằng các thay đổi này được áp dụng trước khi người dùng nhìn thấy nội dung mới.
- Khi bạn cần đảm bảo rằng các tính toán hoặc hiệu chỉnh DOM phải hoàn thành trước khi người dùng tương tác với trang web. Ví dụ, đặt một lắng nghe sự kiện (event listener) trước khi người dùng có thể tương tác với một phần của giao diện.
- Khi bạn muốn tránh việc xuất hiện các hiệu ứng thụ động (layout shifts) khi trình duyệt thực hiện cập nhật. Sử dụng useLayoutEffect() có thể giúp bạn kiểm soát chính xác thời điểm cập nhật giao diện và tránh những hiệu ứng không mong muốn.

### 3.4 useMemo()

#### 3.4.1 useMemo() là gì?

useMemo là một trong những hook cung cấp bởi React dùng để tối ưu hóa hiệu suất của ứng dụng bằng cách lưu giữ kết quả của một hàm tính toán và trả về giá trị đó khi các dependencies của nó không thay đổi. Điều này giúp tránh việc tính toán lại giá trị nếu không cần thiết, đặc biệt là trong trường hợp tính toán tốn thời gian. Cú pháp của useMemo() như sau:

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue ((a, b), [a, b]);
```

Trong đó:

- computeExpensiveValue là hàm mà bạn muốn tính toán một giá trị.
- [a, b] là mảng dependencies. Nếu bất kỳ giá trị nào trong mảng này thay đổi, computeExpensiveValue sẽ được tính toán lại và kết quả mới được trả về.

Dưới đây là một ví dụ cơ bản sử dụng useMemo():

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculationComponent() {
  const [a, setA] = useState(1);
  const [b, setB] = useState(2);

  const result = useMemo(() => {
    console.log('Expensive calculation');
    return a + b;
  }, [a, b]);

  return (
    <div>
      <p>Result: {result}</p>
      <button onClick={() => setA(a + 1)}>Increment A</button>
      <button onClick={() => setB(b + 1)}>Increment B</button>
    </div>
  );
}

export default ExpensiveCalculationComponent;
```

Trong ví dụ này, chúng ta sử dụng useMemo để tính toán tổng của a và b mỗi khi giá trị của a hoặc b thay đổi. Việc tính toán chỉ xảy ra khi có sự thay đổi trong a hoặc b, và giá trị result được lưu giữ và không tính toán lại khi component render lại với những thay đổi không liên quan. Điều này giúp cải thiện hiệu suất của ứng dụng.

#### 3.4.2 Khi nào thì nên sử dụng useMemo()

- Tính toán phức tạp: Khi bạn có một biểu thức tính toán hoặc một hàm mà việc tính toán mất nhiều thời gian hoặc tài nguyên, và bạn muốn tránh việc tính toán lại nó mỗi khi component render lại. useMemo giúp lưu giữ kết quả tính toán trước đó và chỉ tính toán lại khi các dependencies thay đổi.
- Tối ưu hóa các kết quả trung gian: Khi bạn có chuỗi các tính toán dựa trên các kết quả trung gian và bạn muốn tối ưu hóa việc tính toán lại chúng. useMemo cho phép bạn lưu giữ các kết quả trung gian và tính toán lại chỉ khi cần thiết.
- Lưu giữ giá trị tính toán: Khi bạn muốn lưu giữ giá trị tính toán cho một số dữ liệu và tránh việc tính toán lại chúng khi component render lại. Điều này thường xảy ra khi bạn làm việc với dữ liệu truy vấn hoặc tính toán đặc biệt mà không thay đổi thường xuyên.

### 3.5 useCallback()

#### 3.5.1 useCallback() là gì?

useCallback là một hook trong React được sử dụng để tối ưu hóa hiệu suất của ứng dụng bằng cách lưu giữ một hàm callback và chỉ tính toán lại nó khi các dependencies của nó thay đổi. Điều này đặc biệt hữu ích khi bạn truyền các hàm callback làm props cho các component con, và bạn không muốn component render lại mỗi khi một hàm callback bên ngoài thay đổi. Cú pháp sử dụng useCallback như sau:

```javascript
const memoizedCallback = useCallback(() => {
  // Hàm callback của bạn ở đây
}, [dependencies]);
```

Trong đó:

- Hàm callback của bạn nằm trong hàm useCallback.
- [dependencies] là mảng dependencies. Hàm callback sẽ chỉ được tính toán lại khi các giá trị trong mảng này thay đổi.

Dưới đây là một ví dụ cơ bản sử dụng useCallback():

```javascript
import React, { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]);

  return (
    <div>
      <p>Count: {count}</p>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

function ChildComponent({ onClick }) {
  return (
    <button onClick={onClick}>Increment Count</button>
  );
}

export default ParentComponent;
```

Trong ví dụ này, chúng ta sử dụng useCallback để bọc hàm handleClick. Hàm handleClick chỉ tính toán lại khi count thay đổi, nghĩa là khi dependencies của nó thay đổi. Điều này đảm bảo rằng việc truyền hàm callback không làm cho component render lại mỗi khi nó được truyền.

#### 3.5.2 Khi nào thì nên sử dụng useCallback()

- Tránh re-render không cần thiết: Khi bạn truyền một hàm vào một component con thông qua props và hàm này không cần phải re-render mỗi khi component cha re-render. Bằng cách sử dụng useCallback(), bạn có thể tránh việc tạo ra một phiên bản mới của hàm mỗi khi component cha re-render.
- Sử dụng trong mảng dependencies của useEffect(): Khi bạn sử dụng useEffect() và muốn tránh việc gọi lại useEffect() mỗi khi một hàm trong mảng dependencies thay đổi. Sử dụng useCallback() để bọc hàm và đưa nó vào mảng dependencies sẽ giúp bạn kiểm soát được việc kích hoạt useEffect().
- Tránh việc tạo lại hàm trong render: Khi bạn muốn tránh việc tạo lại hàm mỗi khi component re-render, đặc biệt là trong các component lớn hoặc có hiệu suất đòi hỏi. Bằng cách sử dụng useCallback(), bạn có thể đảm bảo rằng hàm sẽ không bị tạo lại mỗi lần component re-render.
- Tránh việc tạo lại hàm trong một sự kiện như onClick, onChange, và các sự kiện khác: Sử dụng useCallback() để bọc hàm xử lý sự kiện sẽ giúp tối ưu hóa hiệu suất và tránh việc gọi lại hàm không cần thiết.

### 3.6 useRef()

#### 3.6.1 useRef() là gì?

useRef là một hook trong React được sử dụng để tạo một tham chiếu (reference) đến một phần tử DOM hoặc để lưu trữ các giá trị mà bạn muốn duy trì giữa các chu kỳ render của component mà không gây ra việc render lại khi giá trị thay đổi. Cú pháp sử dụng useRef như sau:

```javascript
import React, { useRef } from 'react';

function MyComponent() {
  // Tạo một tham chiếu đến phần tử DOM
  const myElementRef = useRef();

  // Tạo một tham chiếu để lưu trữ giá trị
  const myValueRef = useRef(initialValue);

  // Sử dụng tham chiếu đến phần tử DOM
  return <div ref={myElementRef}>My Element</div>;

  // Sử dụng tham chiếu để lưu trữ giá trị
  myValueRef.current = newValue;
}
```

Dưới đây là ví dụ cơ bản sử dụng useRef():

```javascript
import React, { useRef, useState, useEffect } from 'react';

function ExampleComponent() {
  const countRef = useRef(0);
  const [count, setCount] = useState(0);

  const incrementCount = () => {
    // Sử dụng `countRef` để lưu trữ giá trị không gây ra việc render lại
    countRef.current = countRef.current + 1;

    // Sử dụng `setCount` để cập nhật giá trị trong state và gây ra việc render lại
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count (state): {count}</p>
      <p>Count (ref): {countRef.current}</p>
      <button onClick={incrementCount}>Increment Count</button>
    </div>
  );
}

export default ExampleComponent;
```

Trong ví dụ này, chúng ta tạo countRef, một tham chiếu để lưu trữ giá trị count mà không gây ra việc render lại. Khi nút "Increment Count" được nhấn, chúng ta tăng giá trị trong countRef.current mà không làm thay đổi trạng thái state. Trong khi đó, giá trị trong trạng thái state count được cập nhật và gây ra việc render lại.

#### 3.6.2 Khi nào thì nên sử dụng useRef()

Ví dụ phía trên là trường hợp căn bản của việc sử dụng useRef() để hạn chế component bị re-render. Ngoài ra useRef() còn được cân nhắc sử dụng trong rất nhiều trường hợp khác:

Truy cập và thao tác với phần tử DOM: useRef thường được sử dụng để truy cập và thao tác với các phần tử DOM trong React. Nếu bạn cần thực hiện các thay đổi trực tiếp trên phần tử DOM, như thiết lập trạng thái focus, thay đổi kích thước hoặc vị trí của phần tử, thì useRef là sự lựa chọn phù hợp.

```javascript
import React, { useRef } from 'react';

function DOMAccessExample() {
  const inputRef = useRef(null);

  const focusInput = () => {
    // Sử dụng tham chiếu để focus vào phần tử input
    inputRef.current.focus();
  };

  const resetInput = () => {
    // Sử dụng tham chiếu để xóa giá trị của phần tử input
    inputRef.current.value = '';
  };

  return (
    <div>
      <input type="text" ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
      <button onClick={resetInput}>Reset Input</button>
    </div>
  );
}

export default DOMAccessExample;
```

Ở ví dụ trên:

- Chúng ta sử dụng useRef để tạo inputRef, một tham chiếu đến phần tử input.
- Bên trong component, chúng ta có hai nút: "Focus Input" và "Reset Input".
- Khi nút "Focus Input" được nhấn, hàm focusInput sử dụng inputRef.current.focus() để focus vào phần tử input, làm cho con trỏ trên trình duyệt di chuyển vào phần tử đó.
- Khi nút "Reset Input" được nhấn, hàm resetInput sử dụng inputRef.current.value = '' để xóa giá trị của phần tử input.

Lưu trữ giá trị state trước đó (previous state): Nếu bạn muốn lưu trữ các giá trị mà không muốn gây ra việc render lại component, useRef rất hữu ích. Bạn có thể sử dụng useRef để duy trì giá trị mà không thay đổi trạng thái state, và giá trị này có thể tồn tại qua các chu kỳ render.

```javascript
import React, { useState, useEffect, useRef } from 'react';

function PreviousStateExample() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    // Lưu trạng thái trước của biến count
    prevCountRef.current = count;
  }, [count]);

  const incrementCount = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <p>Previous Count: {prevCountRef.current}</p>
      <button onClick={incrementCount}>Increment Count</button>
    </div>
  );
}

export default PreviousStateExample;
```

### 3.7 useContext()

#### 3.7.1 useContext() là gì?

useContext là một hook trong React được sử dụng để truy cập các giá trị của một Context API. Context API trong React là một cách để truyền dữ liệu từ một thành phần cha đến các thành phần con mà không cần truyền props qua nhiều lớp con trung gian. useContext giúp bạn lấy giá trị từ Context API một cách dễ dàng.

Nguồn: Karn Tech (Medium)

#### 3.7.2 Cách sử dụng useContext()

Định nghĩa một Context: Trước tiên, bạn cần định nghĩa một Context bằng cách sử dụng createContext:

```javascript
import React, { createContext } from 'react';

const MyContext = createContext();
```

Tạo một Provider: Bạn cần tạo một thành phần Provider để đặt giá trị cho Context. Thông thường, bạn sẽ làm điều này tại thành phần cha cao nhất của ứng dụng React:

```javascript
function MyContextProvider(props) {
  const myValue = "Giá trị mẫu từ Context"; // Đặt giá trị bạn muốn chia sẻ
  return (
    <MyContext.Provider value={myValue}>
      {props.children}
    </MyContext.Provider>
  );
}
```

Sử dụng useContext trong thành phần con: Bây giờ bạn có thể sử dụng useContext trong các thành phần con để truy cập giá trị từ Context:

```javascript
import React, { useContext } from 'react';

function MyComponent() {
  const valueFromContext = useContext(MyContext);

  return (
    <div>
      Giá trị từ Context: {valueFromContext}
    </div>
  );
}
```

Kết nối Provider và các thành phần con: Để cung cấp giá trị từ Context cho các thành phần con, hãy đảm bảo rằng bạn đã bao quanh các thành phần con bằng thành phần Provider:

```javascript
function App() {
  return (
    <MyContextProvider>
      <MyComponent />
    </MyContextProvider>
  );
}
```

#### 3.7.3 Khi nào thì nên sử dụng useContext()

- Chia sẻ dữ liệu global: Khi bạn cần truyền dữ liệu hoặc trạng thái từ thành phần cha đến các thành phần con ở nhiều mức độ sâu trong ứng dụng mà không muốn truyền props qua từng lớp con trung gian, useContext là lựa chọn tốt. Điều này giúp làm cho mã nguồn của bạn dễ đọc hơn và giảm độ phức tạp của việc truyền props.
- Xác định trạng thái ứng dụng toàn cục: Nếu bạn cần theo dõi trạng thái toàn cục của ứng dụng, ví dụ: trạng thái đăng nhập, ngôn ngữ hiện tại, hoặc chế độ tối/sáng, useContext có thể giúp bạn quản lý trạng thái này một cách hiệu quả và chia sẻ nó với các thành phần con.
- Giữa các thành phần không liên quan trực tiếp: Khi bạn muốn truyền dữ liệu giữa các thành phần không có mối quan hệ cha con trực tiếp, useContext giúp bạn làm điều này mà không cần truyền props qua từng thành phần trung gian.

### 3.8 useReducer()

#### 3.8.1 useReducer() là gì?

useReducer là một hooks được sử dụng để quản lý trạng thái (state) và hành động (actions) của ứng dụng. useReducer gần giống với useState, tuy nhiên nó thường được sử dụng khi bạn cần quản lý một trạng thái phức tạp hơn và khi trạng thái của bạn có quá nhiều hành động gây ra sự thay đổi.

#### 3.8.2 Cách sử dụng useReducer()

Định nghĩa reducer function: Trước hết, bạn cần xác định một reducer function. Reducer là một hàm nhận vào hai tham số: trạng thái hiện tại và hành động. Nó sẽ trả về một trạng thái mới dựa trên hành động.

```javascript
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
};
```

Sử dụng useReducer trong component: Trong component của bạn, sử dụng useReducer và truyền vào reducer function và trạng thái ban đầu.

```javascript
import React, { useReducer } from 'react';

function Counter() {
  const initialState = { count: 0 };
  const [state, dispatch] = useReducer(reducer, initialState);
  
  // Bây giờ 'state' chứa trạng thái hiện tại và 'dispatch' là một hàm để gửi hành động.
  
  // Ví dụ về việc tăng/giảm giá trị count:
  const increment = () => {
    dispatch({ type: 'INCREMENT' });
  };

  const decrement = () => {
    dispatch({ type: 'DECREMENT' });
  };

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
    </div>
  );
}
```

Trong ví dụ trên, chúng ta sử dụng useReducer để quản lý trạng thái count. Hành động được xác định bởi dispatch sẽ gửi đến reducer function, và trạng thái mới sẽ được tính toán dựa trên hành động mà chúng ta gửi.

Với useReducer, bạn có thể quản lý trạng thái phức tạp hơn và loại bỏ việc trạng thái bị trôi (state drift) bởi việc xác định rõ các hành động có thể xảy ra.

#### 3.8.3 Khi nào nên sử dụng useReducer()

- Quản lý trạng thái phức tạp: Khi bạn cần quản lý một trạng thái phức tạp với nhiều thuộc tính hoặc mô hình dữ liệu, useReducer có thể giúp bạn duyệt qua các action và cập nhật trạng thái một cách dễ dàng hơn so với useState.
- Tránh sự cố lỗi khi cập nhật trạng thái: Khi bạn đang làm việc với trạng thái phức tạp và muốn đảm bảo tính nhất quán trong việc cập nhật trạng thái, useReducer giúp tránh nhầm lẫn và sai sót trong quá trình cập nhật trạng thái.
- Tái sử dụng logic cập nhật trạng thái: Khi bạn muốn tái sử dụng logic cập nhật trạng thái (như xác định xem một item có được thêm vào danh sách hay không) cho nhiều phần của ứng dụng, useReducer giúp bạn trừu tượng hóa logic này thành một reducer và sử dụng lại ở nhiều nơi.

### 3.9 Custom Hooks

Ngoài việc sử dụng các Hooks có sẵn được cung cấp bởi React, bạn cũng có thể tự tạo ra Hooks của riêng mình để phục vụ yêu cầu nhất định nào đó mà các Hooks React cung cấp không đáp ứng được.

Viết Custom Hooks là một cách để bạn tái sử dụng logic và trạng thái logic liên quan trong nhiều component khác nhau. Chúng giúp bạn tránh việc lặp lại code và làm cho mã của bạn trở nên dễ quản lý hơn. Một Custom Hook là một hàm JavaScript bình thường, bắt đầu bằng từ khóa use và nó có thể gọi các hook có sẵn hoặc các Custom Hook khác. Điều quan trọng là Custom Hook không phải là một hook cơ bản mà là một phong cách tổ chức mã của bạn.

Dưới đây là ví dụ về cách tạo một Custom Hooks đơn giản:

```javascript
import { useState, useEffect } from 'react';

function useCounter(initialValue, step) {
  const [count, setCount] = useState(initialValue);

  const increment = () => {
    setCount(count + step);
  };

  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return { count, increment };
}

export default useCounter;
```

Còn đây là cách sử dụng nó:

```javascript
import React from 'react';
import useCounter from './useCounter';

function Counter() {
  const { count, increment } = useCounter(0, 1);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}

export default Counter;
```

Ở đây, chúng ta đã tạo một Custom Hook có tên useCounter, và nó chứa logic cho việc đếm và tăng giá trị. Sau đó, chúng ta sử dụng Custom Hook này trong component Counter để hiển thị giá trị và tạo hành động tăng giá trị.

Custom Hooks có thể giúp bạn tạo ra các module chứa logic riêng biệt, dễ dàng kiểm thử, và có thể sử dụng lại trong nhiều component khác nhau. Điều này giúp tách biệt logic của bạn khỏi giao diện người dùng và làm cho mã của bạn trở nên dễ bảo trì hơn.

## 4. Rule of Hooks

Hooks cũng có những nguyên tắc nhất định bắt chúng ta phải tuân theo để đảm bảo component hoạt động đúng cách và ổn định, hạn chế xảy ra lỗi không mong muốn. Có 3 nguyên tắc trong việc sử dụng Hooks:

- Chỉ được sử dụng Hook bên trong component: Hook không được gọi trong các hàm JavaScript thông thường, các hàm helper, hoặc bất kỳ phạm vi khác ngoài component. Chúng phải được gọi bên trong một component.
- Hook chỉ được gọi ở cấp độ ngang hàng (top level): Hook phải được gọi ở cấp độ ngang hàng của hàm component. Điều này có nghĩa là bạn không nên gọi hook trong các hàm con, các câu lệnh điều kiện, hoặc vòng lặp.
- Hook phải được gọi trong cùng một thứ tự mỗi lần render: Hook phải luôn được gọi trong cùng một thứ tự trong mỗi lần render. Điều này giúp React theo dõi việc cập nhật các hook và đảm bảo rằng chúng sẽ hoạt động đúng cách.

Ví dụ dưới đây mô tả về trường hợp không tuân thủ theo nguyên tắc của Hooks:

```javascript
function MyComponent() {
  if (someCondition) {
    // Không tuân theo Rule of Hooks: Gọi hook trong hàm điều kiện
    useState(0);
  }

  for (let i = 0; i < 5; i++) {
    // Không tuân theo Rule of Hooks: Gọi hook trong vòng lặp
    useEffect(() => {
      // ...
    });
  }

  // Tuân theo Rule of Hooks: Gọi hook ở cấp độ hàng ngang
  const [count, setCount] = useState(0);
  useEffect(() => {
    // ...
  });

  return (
    // ...
  );
}
```

Trong ví dụ trên, gọi hook trong hàm điều kiện hoặc vòng lặp là vi phạm "Rule of Hooks" và có thể gây ra lỗi. Hook phải được gọi ở cấp độ ngang hàng của component và luôn ở cùng một thứ tự.

## 5. Kết luận

React Hooks là một tính năng mạnh mẽ trong React cho phép bạn quản lý trạng thái và hiệu suất một cách dễ dàng trong các component functional. Với Hooks, bạn có thể sử dụng trạng thái, hiệu suất, và các tác vụ phụ thuộc vào trạng thái một cách tự nhiên, tạo ra mã ngắn gọn và dễ đọc hơn. Hooks như useState, useEffect, và useRef đã thay đổi cách chúng ta xây dựng ứng dụng React, làm cho việc phát triển và bảo trì ứng dụng trở nên linh hoạt và hiệu quả hơn.