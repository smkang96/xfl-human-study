## 테스트 실패 정보

아래는 실패한 테스트와 실패한 에러 메시지에 대한 `pytest` 출력입니다.
실패한 테스트 함수는 [여기](./repo/pandas/tests/series/test_analytics.py#L288)서도 보실 수 있습니다.

> <span style="background-color:red">!!</span> 이 예제에는 버그와 상관이 없음에도 불구하고 실패하는 없는 4개의 테스트가 있습니다. 아래 테스트를 실패해도 디버깅에 성공한 것으로 간주하시면 됩니다.
> * pandas/tests/series/test_analytics.py::TestSeriesAnalytics::test_cummin_datetime64
> * pandas/tests/series/test_analytics.py::TestSeriesAnalytics::test_cummax_datetime64
> * pandas/tests/series/test_analytics.py::TestSeriesAnalytics::test_cummin_timedelta64
> * pandas/tests/series/test_analytics.py::TestSeriesAnalytics::test_cummax_timedelta64

```python
============================= test session starts ==============================
platform linux -- Python 3.8.3, pytest-5.4.3, py-1.8.1, pluggy-0.13.1
rootdir: /home/user/BugsInPy/temp/projects/pandas, inifile: setup.cfg
plugins: hypothesis-5.16.0
collected 1 item

pandas/tests/series/test_analytics.py F                                  [100%]

=================================== FAILURES ===================================
_____________ TestSeriesAnalytics.test_bool_diff[input0-output0-1] _____________

self = <pandas.tests.series.test_analytics.TestSeriesAnalytics object at 0x7f4011785940>
input = [False, True, True, False, False]
output = [nan, True, False, True, False], diff = 1

    @pytest.mark.parametrize(
        "input,output,diff",
        [([False, True, True, False, False], [nan, True, False, True, False], 1)],
    )
    def test_bool_diff(self, input, output, diff):
        # boolean series (test for fixing #17294)
        s = Series(input)
>       result = s.diff()

pandas/tests/series/test_analytics.py:291: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
pandas/core/series.py:2478: in diff
    result = algorithms.diff(com.values_from_object(self), periods)
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

arr = array([False,  True,  True, False, False]), n = 1, axis = 0

    def diff(arr, n: int, axis: int = 0):
        """
        difference of n between self,
        analogous to s-s.shift(n)
    
        Parameters
        ----------
        arr : ndarray
        n : int
            number of periods
        axis : int
            axis to shift on
    
        Returns
        -------
        shifted
        """
    
        n = int(n)
        na = np.nan
        dtype = arr.dtype
    
        is_timedelta = False
        if needs_i8_conversion(arr):
            dtype = np.float64
            arr = arr.view("i8")
            na = iNaT
            is_timedelta = True
    
        elif is_bool_dtype(dtype):
            dtype = np.object_
    
        elif is_integer_dtype(dtype):
            dtype = np.float64
    
        dtype = np.dtype(dtype)
        out_arr = np.empty(arr.shape, dtype=dtype)
    
        na_indexer = [slice(None)] * arr.ndim
        na_indexer[axis] = slice(None, n) if n >= 0 else slice(n, None)
        out_arr[tuple(na_indexer)] = na
    
        if arr.ndim == 2 and arr.dtype.name in _diff_special:
            f = _diff_special[arr.dtype.name]
            f(arr, out_arr, n, axis)
        else:
            # To keep mypy happy, _res_indexer is a list while res_indexer is
            #  a tuple, ditto for lag_indexer.
            _res_indexer = [slice(None)] * arr.ndim
            _res_indexer[axis] = slice(n, None) if n >= 0 else slice(None, n)
            res_indexer = tuple(_res_indexer)
    
            _lag_indexer = [slice(None)] * arr.ndim
            _lag_indexer[axis] = slice(None, -n) if n > 0 else slice(-n, None)
            lag_indexer = tuple(_lag_indexer)
    
            # need to make sure that we account for na for datelike/timedelta
            # we don't actually want to subtract these i8 numbers
            if is_timedelta:
                res = arr[res_indexer]
                lag = arr[lag_indexer]
    
                mask = (arr[res_indexer] == na) | (arr[lag_indexer] == na)
                if mask.any():
                    res = res.copy()
                    res[mask] = 0
                    lag = lag.copy()
                    lag[mask] = 0
    
                result = res - lag
                result[mask] = na
                out_arr[res_indexer] = result
            else:
>               out_arr[res_indexer] = arr[res_indexer] - arr[lag_indexer]
E               TypeError: numpy boolean subtract, the `-` operator, is not supported, use the bitwise_xor, the `^` operator, or the logical_xor function instead.

pandas/core/algorithms.py:1963: TypeError
```