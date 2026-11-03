
```
        Py_Dealloc(v44);
    }
    if ( !v34 )
      goto LABEL_205;
    if ( *v29 >= 0 )
    {
      v23 = (*(_QWORD *)v29)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v29);
    }
    if ( *v38 >= 0 )
    {
      v23 = (*(_QWORD *)v38)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v38);
    }
    v45 = v88;
    v88 = (int *)v34;
    if ( v45 )
    {
      if ( *v45 >= 0 )
      {
        v23 = (*(_QWORD *)v45)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v45);
      }
    }
    v46 = *((_QWORD *)off_18000B688 + 19);
    Item_KnownHash = (int *)PyDict_GetItem_KnownHash(*(_QWORD *)off_18000B688, v46, *(_QWORD *)(v46 + 24));
    v48 = Item_KnownHash;
    if ( Item_KnownHash )
    {
      v49 = *Item_KnownHash + 1;
      if ( v49 )
        *v48 = v49;
      v39 = v48;
    }
    else if ( PyErr_Occurred() || (v39 = (int *)sub_180003D40(v46), (v48 = v39) == 0LL) )
    {
      v7 = 17;
      v8 = 2941;
      v4 = (int *)v22;
      goto LABEL_223;
    }
    v50 = 0LL;
    v51 = 0;
    if ( *((_QWORD *)v48 + 1) == PyMethod_Type )
    {
      v50 = (_DWORD *)*((_QWORD *)v48 + 3);
      if ( v50 )
      {
        v39 = (int *)*((_QWORD *)v48 + 2);
        if ( *v50 != -1 )
          ++*v50;
        if ( *v39 != -1 )
          ++*v39;
        if ( *v48 >= 0 )
        {
          v23 = (*(_QWORD *)v48)-- == 1LL;
          if ( v23 )
            Py_Dealloc(v48);
        }
        v51 = 1;
      }
    }
    v85[0] = v34;
    v52 = sub_180004660(v39, &v85[-v51], (unsigned int)(v51 + 1));
    if ( v50 )
    {
      if ( (int)*v50 >= 0 )
      {
        v23 = (*(_QWORD *)v50)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v50);
      }
    }
    if ( !v52 )
    {
      v7 = 17;
      v8 = 2961;
      v4 = (int *)v22;
LABEL_216:
      v5 = v88;
      goto LABEL_217;
    }
    if ( *v39 >= 0 )
    {
      v23 = (*(_QWORD *)v39)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v39);
    }
    v54 = v84;
    v55 = (int *)v52;
    v84 = (int *)v52;
    if ( v54 )
    {
      if ( *v54 >= 0 )
      {
        v23 = (*(_QWORD *)v54)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v54);
      }
    }
    v34 = sub_180004790(v52, 1LL, v53, 0LL);
    if ( !v34 )
    {
      v7 = 18;
      v8 = 2975;
      v4 = (int *)v22;
      goto LABEL_223;
    }
    v58 = *((_QWORD *)off_18000B688 + 31);
    v59 = *(_QWORD *)(v22 + 8);
    if ( v59 == PyLong_Type[0] )
    {
      v60 = *(_QWORD *)(v22 + 16);
      if ( (v60 & 1) != 0 )
      {
        if ( *(_DWORD *)v22 != -1 )
          ++*(_DWORD *)v22;
        v61 = (int *)v22;
        v39 = (int *)v22;
        goto LABEL_114;
      }
      v62 = v60 >= 0x10
          ? (*(__int64 (__fastcall **)(__int64, _QWORD))(PyLong_Type[12] + 16LL))(v22, *((_QWORD *)off_18000B688 + 31))
          : PyLong_FromLongLong(2LL * (int)(*(_DWORD *)(v22 + 24) * (1 - (v60 & 3))), PyLong_Type[0], v58, v57);
    }
    else
    {
      v62 = v59 == PyFloat_Type
          ? PyFloat_FromDouble(v56, PyLong_Type[0], v58, v57)
          : PyNumber_Multiply(v22, *((_QWORD *)off_18000B688 + 31));
    }
    v61 = (int *)v62;
    v39 = (int *)v62;
    if ( !v62 )
    {
      v7 = 18;
      v8 = 2977;
      v4 = (int *)v22;
      goto LABEL_213;
    }
LABEL_114:
    v30 = (int *)sub_180004930(v83, v61, v58);
    if ( !v30 )
    {
      v7 = 18;
      v8 = 2979;
      v4 = (int *)v22;
      goto LABEL_213;
    }
    if ( *v61 >= 0 )
    {
      v23 = (*(_QWORD *)v61)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v61);
    }
    v39 = (int *)PyObject_RichCompare(v34, v30, 2LL);
    if ( !v39 )
    {
      v7 = 18;
      v8 = 2982;
      goto LABEL_209;
    }
    if ( *(int *)v34 >= 0 )
    {
      v23 = (*(_QWORD *)v34)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v34);
    }
    if ( *v30 >= 0 )
    {
      v23 = (*(_QWORD *)v30)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v30);
    }
    IsTrue = v39 == (int *)Py_TrueStruct;
    if ( !(IsTrue | (v39 == (int *)Py_FalseStruct || v39 == (int *)Py_NoneStruct)) )
      IsTrue = PyObject_IsTrue(v39);
    if ( IsTrue = 0 )
    {
      v23 = (*(_QWORD *)v39)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v39);
    }
    if ( !IsTrue )
      goto LABEL_172;
    v39 = (int *)sub_180004790(v55, 0LL, v63, 0LL);
    if ( !v39 )
    {
      v7 = 18;
      v8 = 2992;
      v4 = (int *)v22;
      goto LABEL_223;
    }
    v66 = off_18000B688;
    v67 = *(_QWORD *)(v22 + 8);
    v68 = PyLong_Type[0];
    v69 = *((_QWORD *)off_18000B688 + 31);
    if ( v67 != PyLong_Type[0] )
    {
      if ( v67 == PyFloat_Type )
        v72 = PyFloat_FromDouble(v65, off_18000B688, PyLong_Type[0], v69);
      else
        v72 = PyNumber_Multiply(*((_QWORD *)off_18000B688 + 31), v22);
      goto LABEL_144;
    }
    v70 = *(_QWORD *)(v22 + 16);
    if ( (v70 & 1) == 0 )
    {
      if ( v70 >= 0x10 )
        v72 = (*(__int64 (__fastcall **)(_QWORD, __int64))(PyLong_Type[12] + 16LL))(
                *((_QWORD *)off_18000B688 + 31),
                v22);
      else
        v72 = PyLong_FromLongLong(
                2LL * (int)(*(_DWORD *)(v22 + 24) * (1 - (v70 & 3))),
                off_18000B688,
                PyLong_Type[0],
                v69);
LABEL_144:
      v71 = (int *)v72;
      v30 = (int *)v72;
      if ( !v72 )
      {
        v7 = 18;
        v8 = 2994;
        v4 = (int *)v22;
        goto LABEL_218;
      }
      v66 = off_18000B688;
      goto LABEL_146;
    }
    if ( *(_DWORD *)v22 != -1 )
      ++*(_DWORD *)v22;
    v71 = (int *)v22;
    v30 = (int *)v22;
LABEL_146:
    v34 = sub_1800044A0(v71, v66[30], v68, 0LL);
    if ( !v34 )
    {
      v7 = 18;
      v8 = 2996;
LABEL_209:
      v4 = (int *)v22;
      if ( *v30 >= 0 )
      {
        v23 = (*(_QWORD *)v30)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v30);
      }
      if ( !v34 )
        goto LABEL_216;
      goto LABEL_213;
    }
    if ( *v71 >= 0 )
    {
      v23 = (*(_QWORD *)v71)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v71);
    }
    v30 = (int *)sub_180004930(v83, v34, v73);
    if ( !v30 )
    {
      v7 = 18;
      v8 = 2999;
      v4 = (int *)v22;
      goto LABEL_213;
    }
    if ( *(int *)v34 >= 0 )
    {
      v23 = (*(_QWORD *)v34)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v34);
    }
    v34 = PyObject_RichCompare(v39, v30, 2LL);
    if ( !v34 )
    {
      v7 = 18;
      v8 = 3002;
      goto LABEL_209;
    }
    if ( *v39 >= 0 )
    {
      v23 = (*(_QWORD *)v39)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v39);
    }
    v39 = 0LL;
    if ( *v30 >= 0 )
    {
      v23 = (*(_QWORD *)v30)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v30);
    }
    v75 = v34 == Py_TrueStruct;
    if ( !(v75 | (v34 == Py_FalseStruct || v34 == Py_NoneStruct)) )
      v75 = PyObject_IsTrue(v34);
    if ( v75 = 0 )
      {
        v23 = (*(_QWORD *)v34)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v34);
      }
      goto LABEL_216;
    }
    if ( *(int *)v34 >= 0 )
    {
      v23 = (*(_QWORD *)v34)-- == 1LL;
      if ( v23 )
        Py_Dealloc(v34);
    }
    if ( v75 )
    {
      v76 = sub_1800044A0(v19, *((_QWORD *)off_18000B688 + 30), v74, 1LL);
      if ( !v76 )
      {
        v7 = 19;
        v8 = 3018;
        v4 = (int *)v22;
        goto LABEL_223;
      }
      v77 = (int *)v19;
      v89 = v76;
      v19 = v76;
      if ( *v77 >= 0 )
      {
        v23 = (*(_QWORD *)v77)-- == 1LL;
        if ( v23 )
          Py_Dealloc(v77);
      }
    }
LABEL_172:
    v39 = 0LL;
    v20 = v86 + 1;
    v4 = (int *)v22;
    v86 = v20;
    if ( v20 >= 4 )
      break;
    v2 = a2;
  }
  v78 = *((_QWORD *)off_18000B688 + 32);
  if ( v19 == v78 )
  {
    v79 = Py_TrueStruct;
  }
  else
  {
```
