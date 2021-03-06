---
title: Unit Test 
type: guide
order: 403
---

## Cài đặt

Để thực hiện unit test với Vue, bạn có thể sử dụng bất kì công cụ kiểm thử nào tương thích với hệ thống xây dựng dựa trên module. Tuy nhiên, nếu bạn cần một lời khuyên cụ thể thì chúng tôi khuyến khích sử dụng [Karma](http://karma-runner.github.io). Karma có rất nhiều plugin, bao gồm các plugin cung cấp các tính năng hỗ trợ [Webpack](https://github.com/webpack/karma-webpack) và [Browserify](https://github.com/Nikku/karma-browserify). Để tìm hiểu thông tin cài đặt Karma và các plugin này, hãy xem các tài liệu tương ứng. Bạn cũng có thể bắt đầu bằng cách tham khảo các ví dụ về cấu hình dành cho [Webpack](https://github.com/vuejs-templates/webpack/blob/master/template/test/unit/karma.conf.js) và [Browserify](https://github.com/vuejs-templates/browserify/blob/master/template/karma.conf.js).

## Các assert đơn giản

Bạn không cần phải làm gì đặc biệt để cho component có thể test được. Chỉ cần `export` các tùy chọn thô:

``` html
<template>
  <span>{{ message }}</span>
</template>

<script>
  export default {
    data () {
      return {
        message: 'hello!'
      }
    },
    created () {
      this.message = 'bye!'
    }
  }
</script>
```

sau đó `import` Vue và component, thế là bạn đã có thể thực hiện các assertion thông dụng:

``` js
// Import Vue và component cần test
import Vue from 'vue'
import MyComponent from 'path/to/MyComponent.vue'

// Đây là một số lệnh test sử dụng Jasmine 2.0
// Tất nhiên bạn có thể sử dụng bất kì thư viện test nào
// bạn muốn
describe('MyComponent', () => {
  // Kiểm tra các tùy chọn của component 
  it('has a created hook', () => {
    expect(typeof MyComponent.created).toBe('function')
  })

  // Tính toán giá trị trả về của các hàm trong tùy chọn
  // của component
  it('sets the correct default data', () => {
    expect(typeof MyComponent.data).toBe('function')
    const defaultData = MyComponent.data()
    expect(defaultData.message).toBe('hello!')
  })

  // Kiểm tra đối tượng component khi `mount`
  it('correctly sets the message when created', () => {
    const vm = new Vue(MyComponent).$mount()
    expect(vm.message).toBe('bye!')
  })

  // `mount` một đối tượng và kiểm tra kết quả render
  it('renders the correct message', () => {
    const Ctor = Vue.extend(MyComponent)
    const vm = new Ctor().$mount()
    expect(vm.$el.textContent).toBe('bye!')
  })
})
```

## Viết component có thể test được

Kết quả render của một component được quyết định phần lớn là bởi các `props` nó nhận được. Nếu kết quả render của component chỉ phụ thuộc vào props, việc test sẽ trở nên rất dễ dàng, tương tự như việc kiểm tra giá trị trả về của một hàm thuần ([pure function](https://en.wikipedia.org/wiki/Pure_function)) với các tham số khác nhau. Thử lấy một ví dụ đơn giản:

``` html
<template>
  <p>{{ msg }}</p>
</template>

<script>
  export default {
    props: ['msg']
  }
</script>
```

Bạn có thể kiểm tra kết quả render với các prop khác nhau của component này bằng cách sử dụng tùy chọn `propsData`:

``` js
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// helper function that mounts and returns the rendered text
function getRenderedText (Component, propsData) {
  const Ctor = Vue.extend(Component)
  const vm = new Ctor({ propsData: propsData }).$mount()
  return vm.$el.textContent
}

describe('MyComponent', () => {
  it('renders correctly with different props', () => {
    expect(getRenderedText(MyComponent, {
      msg: 'Hello'
    })).toBe('Hello')

    expect(getRenderedText(MyComponent, {
      msg: 'Bye'
    })).toBe('Bye')
  })
})
```

## Assert cập nhật không đồng bộ

Do Vue [thực hiện cập nhật DOM một cách không đồng bộ](reactivity.html#Async-Update-Queue), đối với các thay đổi trên DOM xảy ra do thay đổi state, bạn sẽ phải thực hiện assert trong hàm callback `Vue.nextTick`:

``` js
// Kiểm tra HTML được tạo ra sau khi cập nhật state
it('updates the rendered message when vm.message updates', done => {
  const vm = new Vue(MyComponent).$mount()
  vm.message = 'foo'

  // đợi một "tick" (tích tắc) sau khi state thay đổi
  // rồi mới thực hiện assert các cập nhật trên DOM
  Vue.nextTick(() => {
    expect(vm.$el.textContent).toBe('foo')
    done()
  })
})
```
