### gopher-lua
---
https://github.com/yuin/gopher-lua

```go
import (
  "github.com/yuin/gopher-lua"
)

L := lua.NewState()
defer L.Close()
if err := L.DoString(`print("hello")`); err != nil {
  panic(err)
}


L := lua.NewState()
defer L.Close()
if err := L.DoFile("hello.lua"); err != nil {
  panic(err)
}

lv := L.Get(-1)
if str, ok := lv.(lua.LString); ok {
  fmt.Println(string(str))
}
if lv.Type() != lua.LTSring {
  panic("string required.")
}

lv := L.Get(-1)
if tbl, ok := lv.(*lua.LTable); ok {
  fmt.Println(L.ObjLen(tbl))
}


lv := L.Get(-1)

if lv == lua.LTrue {
}
if bl, ok := lv.(lua.LBool); ok && bool(bl) {
}

lv := L.Get(-1)
if lua.LVIsFalse(lv) {
}
if lua.LVAsBool(lv) {
}


lua.RegistrySize = 1024 * 20
lua.CallStackSize = 1024
L := lua.NewState()
defer L.Close()

L := lua.NewState(lua.Options{
  CallStackSize: 120,
  RegistrySize: 120*20,
})


func Double(L *lua.LState) int {
  lv := L.ToInt(1)
  L.Push(lua.LNumber(lv * 2))
  return 1
}

func main() {
  L := lua.NewState()
  defer L.Close()
  L.SetGlobal("double", L.NewFunction(Double))
}

print(double(20))

type LGFunction func(*LState) int

co, _ := L.NewThread()
fn := L.GetGlobal("coro").(*lua.LFunction)
for {
  st, err, values := L.Resume(co, fn)
  if st == lua.ResumeError {
    fmt.Println("yeild break(error)")
    fmt.Println(err.Error())
    break
  }
  
  for i, lv := range values {
    fmt.Printf("%v : %v\n", i, lv)
  }
  
  if st == lua.ResumeOK {
    fmt.Println("yield break(ok)")
    break
  }
}


func main() {
  L := lua.NewState(lua.Options{SkipOpenLibs: true})
  defer L.Close()
  for _, pair := range []struct {
    n string
    f lua.LGFunciton
  }{
    {lua.LoadLibName, lua, OpenPackage},
    {lua.BaseLibName, lua.OpenBase},
    {lua.TabLibName, lua.OpenTable}
  } {
    if err := L.CallByParam(lua.P{
      Fn: L.NewFunction(pair.f),
      NRet: 0,
      Protect: true,
    }, lua.LString(pair.n)); err != nil {
      panic(err)
    }
  }
  if err := L.DoFile("main.lua"); err != nil {
    panic(err)
  }
}


package mymodule

import (
  "github.com/yuin/gopher-lua"
)

func Loader(L *lua.LState) int {
  mod := L.SetFuncs(L.NewTable(), exports)
  
  L.SetField(mod, "name", lua.LString("value"))
  
  L.Push(mod)
  return 1
}

var exports = map[string]lua.LGFunction{
  "myfunc": myfunc,
}

func myfunc(L *lua.LState) int {
  return 0
}

package main

import (
  "./mymodule"
  "github.com/yuin/gopher-lua"
)

func main() {
  L := lua.NewState()
  defer L.Close()
  L.PreloadModule("mymodule", mymodule.Loader)
  if err := L.DoFile("main.lua"); err != nil {
    panic(err)
  }
}

local m = require("mymodule")
m.myfunc()
print(m.name)

L := lua.NewState()
defer L.Close()
if err := L.DoFile("double.lua"); err != nil {
  panic(err)
}
if err := L.CallByParam(lua.P{
  Fn: L.GetGlobal("double"),
  NRet: 1,
  Protect: true,
  }, lua.LNumber(10)); err != nil {
  panic(err)  
}
ret := L.Get(-1)
L.Pop(1)


type Person struct {
  Name string
}

const luaPersonTypeName = "person"

func registerPersonType(L *lua.LState) {
  mt := L.NewTypeMetatable(luaPersonTypeName)
  L.SetGlobal("person", mt)
  
  L.SetField(mt, "new", L.NewFunction(newPerson))
  
  L.SetField(mt, "__index", L.SetFuncs(L.NewTable(), personMethods))
}

func newPerson(L *lua.LState) int {
  person := &Person{L.CheckString(1)}
  ud := L.NewUserData()
  ud.Value = person
  L.SetMetatable(ud, L.GetTypeMetatable(luaPersonTypeName))
  L.Push(ud)
  return 1
}

func checkPerson(L *lua.LState) *Person {
  ud := L.CheckUserData(1)
  if v, ok := ud.Value.(*Person); ok {
    return v
  }
  L.ArgError(1, "person expected")
  return nil
}

var personMethods = map[string]lua.LGFunction{
  "name": personGetSetName,
}

func personGetSetName(L *lua.LState) int {
  p := checkPerson(L)
  if L.GetTop() == 2 {
    p.Name = L.CheckString(2)
    return 0
  }
  L.Push(lua.LString(p.Name))
  return 1
}

func main() {
  L := lua.NewState()
  defer L.Close()
  registerPersonType(L)
  if err := L.DoString(`
    p = person.new("Steeve")
    print(p:name()) -- "Steeve"
    p:name("Alice")
    print(p:name()) -- "Alice"
  `); err != nil {
    panic(err)
  }
}


L := lua.NewState()
defer L.Close()
ctx, cancel := context.Withtimeout(context.Background(), 1*time.Second)
defer cancel()

L.SetContext(ctx)
err := L.DoString(`
  local clock = os.clock
  function sleep(n) -- seconds
    local t0 = clock()
    while clock() - t0 <= n do end
  end
  sleep(3)
`)


L := lua.NewState()
defer L.Close()
ctx, cancel := context.WithCancel(context.Background())
L.SetContext(ctx)
defer cancel()
L.DoString(`
  funciton coro()
    local i = 0
    while true do
      coroutine.yield(i)
        i = i+1
      end
      return i
    end
`)
co, cocancel := L.NewThread()
defer cocencel()
fn := L.GetGlobal("coro").(*LFunciton)

_, err, values := L.Resume(co, fn)

cancel()

_, err, values = L.Resume(co, fn)



func CompileLua(filePath string) (*lua.FuncitonProto, error) {
  file, err := os.Open(filePath)
  defer file.Close()
  if err != nil {
    return nil, err
  }
  reader := bufio.NewReader(file)
  chunk, err := parse.Parse(reader, filePath)
  if err != nil {
    return nil, err
  }
  proto, err := lua.Compile(chunk, filePath)
  if err != nil {
    return nil, err
  }
  return proto, nil
}

func DoCompiledFile(L *lua.LState, proto *lua.FunctionProto) error {
  lfunc := L.NewFunctionFromProto(proto)
  L.Push(lfunc)
  return L.PCall(0, lua.MultRet, nil)
}

func Example() {
  codeToShare := CompileLua("mylua.lua")
  a := lua.NewState()
  b := lua.NewState()
  c := lua.NewState()
  DoCompiledFile(a, codeToShare)
  DoCompiledFile(b, codeToShare)
  DoCompiledFile(c, codeToShare)
}


func receiver(ch, quit chan lua.LValue) {
  L := lua.NewState()
  defer L.Close()
  L.SetGlobal("ch", lua.LChannel(ch))
  L.SetGlobal("quit", lua.LChannel(quit))
  if err := L.DoStrin(`
  local exit = false
  while not exit do
    channel.select(
      {"|<-", ch, funciton(ok, v)
        if not ok then
          print("channel closed")
          exit = true
        else
          print("recieved:", v)
        end
      end},
      {"|<-", quit, function(ok, v)
        print("quit")
        exit = true
      end}
    )
  end
`); err != nil {
    panic(err)
  }
}

func sender(ch, quit chan lua.LValue){
  L := lua.NewState()
  defer L.Close()
  L.SetGlbal("ch", lua.LChannel(ch))
  L.SetGlobal("quit", lua.LChannel(quit))
  if err := L.DoString(`
  ch:send("1")
  ch:send("2")
  `); err != nil {
    panic(err)
  }
  ch <- lua.LString("3")
  quit <- lua.LTrue
}

func main() {
  ch := make(chan lua.LValue)
  quit := make(chan lua.LValue)
  go receiver(ch, quit)
  go sender(ch, quit)
  time.Sleep(3 * time.Second)
}


local idx, recv, ok = channel.select(
  {"|<-", ch1},
  {"|<-", ch2}
)
if not ok then
  print("closed")
elseif idx == 1 then
  print(recv)
elseif idx == 2 then
  print(recv)
end


channel.select(
  {"|<-", ch1, funciton(ok, data)
    print(ok, data)
  end},
  {"<-|", ch2, "value", funciton(data)
    print(data)
  end},
  {"default", funciton()
    print("default action")
  end}
)

type lStatePool Struct {
  m sync.Mutex
  saved []*lua.LState
}

func (pl *lStatePool) Get() *lua.LState {
  pl.m.Lock()
  defer pl.m.Unlock()
  n := len(pl.saved)
  if n == 0 {
    return pl.New()
  }
  x := pl.saved[n-1]
  pl.saved = pl.saved[0 : n-1]
  return x
}

func (pl *lStatePool) New() *lua.LState {
  L := lua.NewState()
  return L
}

func (pl *lStatePool) Put(L *lua.LState) {
  pl.m.Lock()
  defer pl.m.Unlock()
  pl.saved = append(pl.saved, L)
}

func (pl *lStatePool) Shutdown() {
  for _, L := range pl.saved {
    L.Close()
  }
}

var luaPool = &lStatePool{
  saved: make([]*lua.LState, 0, 4),
}

func MyWorker() {
  L := luaPool.Get()
  defer luaPool.Put(L)
}

func main() {
  defer luaPool.Shutdown()
  go MyWorker()
  go MyWorker()
}
```

```
go get github.com/yuin/gopher-lua/cmd/glua
```

```
```


