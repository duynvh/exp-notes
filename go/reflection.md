# Reflection

1. **Reflection là gì?**
- Dùng để inspect variable và biết được type, value của nó trong run-time.
- Được implement trong package `reflect` trong Go.

2. **Khi nào cần dùng reflection?**
- Thông thường thì chúng ta sẽ biết được type và được định nghĩa trong app thì đã có thể biết được ở trong compile time rồi. Nhưng không phải lúc nào thì điều này cũng đúng, và lúc đó thì reflection cần xài tới.
- Đối với những task như viết một cái ORM thì bạn sẽ nghĩ ngay tới reflection. Để có thể build được một query builder, thì input thông thường là không thể biết trước được type (interface{}). Lúc này thì bạn chỉ có thể biết được type của nó ở run-time (vì nếu muốn định nghĩa hết ở compile time thì sẽ phải implement lại cho từng type của model 🥲), convert value ra type tương ứng, ... thì reflection quá tuyệt vời để xài rồi. 

3. **Example**
- Giờ để làm rõ cho ý ở trên chúng ta coi qua ví dụ này nhá:
```go
package main

import (  
    "fmt"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(o order) string {  
    i := fmt.Sprintf("insert into order values(%d, %d)", o.ordId, o.customerId)
    return i
}

func main() {  
    o := order{
        ordId:      1234,
        customerId: 567,
    }
    fmt.Println(createQuery(o))
}
```
- Như ở vd trên thì có thể thấy, hàm `createQuery` phải biết được type của input là `order` ở compile time.
- Nhưng vấn đề là, giờ cần tạo query cho một struct khác, thì lại phải implement một function khác -> nooooo
- Vậy để có thể tạo ra một function dynamic thì chúng ta đổi lại thành thế này
```go
package main

type order struct {  
    ordId      int
    customerId int
}

type employee struct {  
    name string
    id int
    address string
    salary int
    country string
}

func createQuery(q interface{}) string {  
}

func main() {

}
```
- Và để vẫn cho ra được kết quả như version đầu tiên, chúng ta cần dùng tới reflect
- Đầu tiên, chúng ta xài thử `reflect.TypeOf` và `reflect.ValueOf` để xem công dụng của nó đã nhé
```go
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    t := reflect.TypeOf(q)
    v := reflect.ValueOf(q)
    fmt.Println("Type ", t)
    fmt.Println("Value ", v)


}
func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

}
```
- Kết quả là:
> Type  main.order  
> Value  {456 56}  

- Có thể thấy qua ví dụ trên, thì
	- `reflect.TypeOf` sẽ trả về `reflect.Type` chứa type cụ thể của biến tham số interface{}
	- `reflect.ValueOf` sẽ trả về `reflect.Value` chứa underlying value của biến tham số interface{}
- Tiếp theo sẽ là một type quan trọng nữa trong `reflect` package là `reflect.Kind`
```go
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    t := reflect.TypeOf(q)
    k := t.Kind()
    fmt.Println("Type ", t)
    fmt.Println("Kind ", k)


}
func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

}
```
- Kết quả:
> Type  main.order  
> Kind  struct  

- Có thể thấy `Kind` để thể hiện type là loại nào (user defined hay primitive)
- Tiếp đến là `NumField()` trả về số lượng field của struct và `Field(i int)` để trả về `reflect.Value` của field thứ `i` tương ứng.
```go
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

func createQuery(q interface{}) {  
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        v := reflect.ValueOf(q)
        fmt.Println("Number of fields", v.NumField())
        for i := 0; i < v.NumField(); i++ {
            fmt.Printf("Field:%d type:%T value:%v\n", i, v.Field(i), v.Field(i))
        }
    }

}
func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)
}
```
- Kết quả của đoạn code trên
> Number of fields 2  
> Field:0 type:reflect.Value value:456  
> Field:1 type:reflect.Value value:56  

- Và method `Int()`, `String()` để extract `reflect.Value` thành `int64` hay `string` tương ứng.

- Và ví dụ hoàn chỉnh để viết dynamic func `createQuery`

```go
package main

import (  
    "fmt"
    "reflect"
)

type order struct {  
    ordId      int
    customerId int
}

type employee struct {  
    name    string
    id      int
    address string
    salary  int
    country string
}

func createQuery(q interface{}) {  
    if reflect.ValueOf(q).Kind() == reflect.Struct {
        t := reflect.TypeOf(q).Name()
        query := fmt.Sprintf("insert into %s values(", t)
        v := reflect.ValueOf(q)
        for i := 0; i < v.NumField(); i++ {
            switch v.Field(i).Kind() {
            case reflect.Int:
                if i == 0 {
                    query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
                } else {
                    query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                    query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
                } else {
                    query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
                }
            default:
                fmt.Println("Unsupported type")
                return
            }
        }
        query = fmt.Sprintf("%s)", query)
        fmt.Println(query)
        return

    }
    fmt.Println("unsupported type")
}

func main() {  
    o := order{
        ordId:      456,
        customerId: 56,
    }
    createQuery(o)

    e := employee{
        name:    "Naveen",
        id:      565,
        address: "Coimbatore",
        salary:  90000,
        country: "India",
    }
    createQuery(e)
    i := 90
    createQuery(i)

}
```
- Output
> insert into order values(456, 56)  
> insert into employee values("Naveen", 565, "Coimbatore", 90000, "India")  
> unsupported type  

- Tiếp theo, thêm một bước nữa để có thể có một output xịn hơn kiểu như
> insert into order(ordId, customerId) values(456, 56) 

thì chúng ta cần code thêm một chút
```go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

type employee struct {
	name    string
	id      int
	address string
	salary  int
	country string
}

func createQuery(q interface{}) {
	if reflect.ValueOf(q).Kind() == reflect.Struct {
		t := reflect.TypeOf(q)
		query := fmt.Sprintf("insert into %s(", t.Name())

		fields := reflect.VisibleFields(t)
		for i, field := range fields {
			if i == 0 {
				query = fmt.Sprintf("%s%s", query, field.Name)
			} else {
				query = fmt.Sprintf("%s, %s", query, field.Name)
			}
		}

		query = fmt.Sprintf("%s) values(", query)

		v := reflect.ValueOf(q)
		for i := 0; i < v.NumField(); i++ {
			switch v.Field(i).Kind() {
			case reflect.Int:
				if i == 0 {
					query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
				} else {
					query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
				}
			case reflect.String:
				if i == 0 {
					query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
				} else {
					query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
				}
			default:
				fmt.Println("Unsupported type")
				return
			}
		}
		query = fmt.Sprintf("%s)", query)
		fmt.Println(query)
		return

	}
	fmt.Println("unsupported type")
}

func main() {
	o := order{
		ordId:      456,
		customerId: 56,
	}
	createQuery(o)

	e := employee{
		name:    "Naveen",
		id:      565,
		address: "Coimbatore",
		salary:  90000,
		country: "India",
	}
	createQuery(e)
	i := 90
	createQuery(i)

}
```

4. **Có nên sử dụng reflection không?**
- Trong những case thông dụng thì ko nên dùng reflection, Rob Pike có phát biểu:
> _Clear is better than clever. Reflection is never clear._

- Reflect rất mạnh và là một khái niệm nâng cao trong Go, nên được sử dụng cẩn thận. Và rất khó để viết code clear và dễ maintain với reflection nhá.

**Resources:**
- https://golangbot.com/reflection/
- https://www.geeksforgeeks.org/reflection-in-golang/
- https://pkg.go.dev/reflect