### 序列化中操作字段的方法

常用场景: 图片字段序列化完整路径(改变http前缀为https等等一些字段的附加操作)

```python
class RockDetailSerializer(serializers.ModelSerializer):
    """
    详情序列化
    """

    detail = serializers.SerializerMethodField(read_only=True)
    area_detail = serializers.SerializerMethodField(read_only=True)
    pol_type = serializers.SerializerMethodField(read_only=True)
    lit_des = serializers.SerializerMethodField(read_only=True)
    well_name = serializers.SerializerMethodField(read_only=True)

    class Meta:
        model = Rock
        fields = ('detail', 'area_detail', 'pol_type', 'lit_des', 'depth', 'well_name')

    def get_detail(self, obj):
        try:
            return {'id': obj.id, 'image': obj.image.url, 'lit_com': obj.lit_com, 'multiple': obj.multiple,
                    'color': obj.color, 'pal_fea': obj.pal_fea, 'lit_fea': obj.lit_fea, 'por_fea': obj.por_fea}
        except:
            return None

    def get_area_detail(self, obj):
        try:
            add_obj = obj.area_detail
            return str(add_obj.parent_categry.region) + '-' + str(add_obj.region)
        except:
            return None

    def get_pol_type(self, obj):
        try:
            return obj.pol_type.pol_type
        except:
            return None

    def get_lit_des(self, obj):
        try:
            return obj.lit_des.lit_des
        except:
            return None

    def get_well_name(self, obj):
        try:
            return obj.area_detail.parent_categry.parent_categry.region
        except:
            return None
```