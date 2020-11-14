账号：L-weiqiyuan
密码：y5sctjly%scjfh
密码：y5sctjly%scjfh
密码：y5sctjly%scjfh

7003: tapadmin/admin

export IS_MOCK_LOGIN=1
export FLASK_DEBUG=1




lsof -i:5000
```
op-web/hwop/api.py 4186 
class _AdminRoles(MethodViewBase):
    url = "/admin/roles"    
    def post(self, role_name, role_desc, perm):
    def get(self, offset, limit):

op-web/hwop/modules/user/views.py 132
@perms_required("user_management")
class RolesListMethodView(api._AdminRoles):
    def get(self, offset, limit):
        role_list, total = RoleManager.get_role_list(offset, limit)
        return dict(total=total, items=role_list)
    def post(self, role_name, role_desc, perm, manage_role=None):
        role = RoleManager.add_role(role_name, role_desc, perm, manage_role)
        return role
        
op-web/hwop/modules/user/main.py 697
class RoleManager(object):
    @classmethod
    def add_role(cls, role_name, role_desc, perm, manage_role=None):
```

