# target-manager


## Build

* [![Build Status](https://travis-ci.org/go-rut/target_manager.png)](https://travis-ci.org/go-rut/target_manager)

## Sample

### Test Config

[target.conf.sample](target.conf.sample)

```json
{
  "target_demensions": [
    {
      "target_name": "test1",
      "demensions": []
    },
    {
      "target_name": "test2",
      "demensions": [
        {
          "target_key": "test2-key1",
          "description": "for-test2-key1"
        },
        {
          "target_key": "test2-key2",
          "description": "for-test2-key2"
        }
      ]
    }
  ]
}
```

### Init Config

```go
var (
    manager = tm.NewManager()
)

func TestInitTargetDemensions(t *testing.T) {
    manager.InitFilters(CONFIG_FILE, tm.InitFiltersTypeFromFile)
    return
}
```


### TEST

[manager_test.go](manager_test.go)

```go
func TestTarget(t *testing.T) {

    Convey("get target name's demensions", t, func() {
        Convey("when xxxx not in map", func() {
            Convey("will xxxx not found", func() {
                filtered, e := manager.Compare("xxxx", nil, nil)
                So(e, ShouldNotBeNil)
                So(e, ShouldEqual, tm.ERR_TARGET_NAME_NOT_EXIST)
                So(filtered, ShouldBeTrue)
            })
        })
        Convey("when test1 in map", func() {
            Convey("will get test1's dememsions", func() {
                _, e := manager.Compare("test1", nil, nil)
                So(e, ShouldBeNil)
            })
        })
    })

    Convey("target's dememsions not exist", t, func() {
        Convey("when target values is not nil", func() {
            Convey("will get filtered false", func() {
                filtered, e := manager.Compare("test1", tm.TargetValues{"": ""}, nil)
                So(e, ShouldBeNil)
                So(filtered, ShouldBeFalse)
            })
        })
        Convey("when compare values is not nil", func() {
            Convey("will get test1's dememsions", func() {
                filtered, e := manager.Compare("test1", nil, tm.CompareValues{"": ""})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeFalse)
            })
        })
    })

    Convey("target's dememsions exist", t, func() {
        Convey("when compare values' key not exist", func() {
            Convey("will get error: invalid dememsion", func() {
                filtered, e := manager.Compare("test2", nil, tm.CompareValues{"": ""})
                So(e, ShouldNotBeNil)
                So(e, ShouldEqual, tm.ERR_INVALID_DEMEMSION)
                So(filtered, ShouldBeTrue)

                filtered, e = manager.Compare("test2", tm.TargetValues{"": ""}, tm.CompareValues{"": ""})
                So(e, ShouldNotBeNil)
                So(e, ShouldEqual, tm.ERR_INVALID_DEMEMSION)
                So(filtered, ShouldBeTrue)
            })
        })
        Convey("when traget values not equal compare values", func() {
            Convey("will get filtered true", func() {
                filtered, e := manager.Compare("test2",
                    tm.TargetValues{"": ""},
                    tm.CompareValues{"test2-key1": ""})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeTrue)

                filtered, e = manager.Compare("test2",
                    tm.TargetValues{"test2-key1": 1},
                    tm.CompareValues{"test2-key1": ""})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeTrue)

                filtered, e = manager.Compare("test2",
                    tm.TargetValues{"test2-key1": "1"},
                    tm.CompareValues{"test2-key1": "1",
                        "test2-key2": 0})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeTrue)
            })
        })
        Convey("when traget values equals compare values", func() {
            Convey("will get filtered true", func() {
                filtered, e := manager.Compare("test2",
                    tm.TargetValues{"test2-key1": ""},
                    tm.CompareValues{"test2-key1": ""})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeFalse)

                filtered, e = manager.Compare("test2",
                    tm.TargetValues{"test2-key1": 1},
                    tm.CompareValues{"test2-key1": 1})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeFalse)

                filtered, e = manager.Compare("test2",
                    tm.TargetValues{"test2-key1": "1", "test2-key2": 0},
                    tm.CompareValues{"test2-key1": "1", "test2-key2": 0})
                So(e, ShouldBeNil)
                So(filtered, ShouldBeFalse)
            })
        })
    })
    return
}

```
