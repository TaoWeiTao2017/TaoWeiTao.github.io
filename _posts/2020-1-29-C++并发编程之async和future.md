# 简介
支持并发编程是C++11增加的重要特性，新特性是通过`<thread>`库提供并发编程的，该库不仅提供了接近底层的API，还提供了async和future等高级封装的接口。
# async和future
async支持另起一线程进行异步操作，future可以获取异步操作返回的结果
## async
async有三种标准调用形式：
1. future async(std::launch::async, F func, args)
* 以async的方式执行func，如果没有等待future，主线程退出前调用
* 对调用的future调用get()或者wait()
* 如果返回结果是shared_future,则最后一个指向的对象销毁时，线程才会结束
2. future async(std::launch::deferred, F func, args)
* 形成一个推迟任务，当对返回的future调用get()或者wait(),任务才会启动，如果不调用get()或者wait()，则任务不会启动
3. future async(F func, args)
* 让系统自己选择发射策略。可能是std::launch::async，也可能是std::launch::deferred
## future和shared_future
future和shared_future的关系类似于两个智能指针之间的关系
### future
* future只支持调用一次get(),返回一个指向返回值的引用
* future不提供拷贝构造函数和赋值运算符

### shared_future
* get()可以多次调用，不会令状态失效
* get()是个const函数，返回一个const引用指向”存储于shared state“的值
* 提供拷贝构造函数和赋值运算符

# example
注： 此例子来自于侯捷翻译的C++标准库第2版
## future 和async
```
    string func(void){
        cout << "thread id is : " << hex << this_thread::get_id() << endl;
        return string("test");
    }

    int main(){
        future<string> result(async(func)); 
        cout << "main thread id is : " << hex << this_thread::get_id() << endl;
        string res = result.get();
        cout << "result: " << res << endl;

        return 0;
    }
```
输出结果：
```
    AppledeiMac:thread apple$ g++ -std=c++17 async.cc -o async
    AppledeiMac:thread apple$ ./async 
    main thread id is : 0x111719dc0
    thread id is : 0x700001537000
    result: test
```

## async和shared_future
```
    int queryNumber(){
        cout << "read number: ";
        int num =0;
        cin >> num;
        return num;
    }

    void doSomething(char c, shared_future<int> f){
        int num = f.get();
        for(int i =0; i < num; ++i){
            this_thread::sleep_for(chrono::milliseconds(100));
            cout.put(c).flush();
        }
    }

    int main(){
        shared_future<int> f =async(queryNumber);
        auto f1 = async(launch::async, doSomething, '.', f);
        auto f2 = async(launch::async, doSomething, '+', f);
        auto f3 = async(launch::async, doSomething, '*', f);

        f1.get();
        f2.get();
        f3.get();

        cout << "\n done" << endl;
    }
```
输出结果：
```
    AppledeiMac:thread apple$ ./sharedfuture1 
    read number: 5
    +*.+*.+.*+.*+*.
    done
```
