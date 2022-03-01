```python
groManagement:
    1999-08-01:     #开始时间
        CropCalendar:
            crop_name: wheat
            vatiety_name: winter-wheat
            crop_start_date: 1999-09-15
            crop_start_type: sowing    #sowing/emergence
            crop_end_date:      #收获日期
            crop_end_type: maturity    #maturity/harvest
            max_duration: 300    #没有写收货日期时，将按照crop_start_date + max_duration计算
         
```

