# c++ mangling
- mangling��Ϊ��ʹ�ṹ�壬���ݽṹ���࣬������ͨ�����Ӻͱ��룬�����һЩ�������Ϣ��
- ������
```cpp
int  f (void) { return 1; }
int  f (int)  { return 0; }
void g (void) { int i = f(), j = f(0); }
```

These are distinct functions, with no relation to each other apart from the name. The C++ compiler therefore will encode the type information in the symbol name, the result being something resembling:

```cpp
int  __f_v (void) { return 1; }
int  __f_i (int)  { return 0; }
void __g_v (void) { int i = __f_v(), j = __f_i(0); }
```

- ��������
```cpp
namespace wikipedia 
{
       class article 
          {
                 public:
                       std::string format (void); 
                                /* = _ZN9wikipedia7article6formatEv */                //��ͷΪ_Z��Ƕ��N���������ı�ʾ������9���Դ�����

                                      bool print_to (std::ostream&); 
                                               /* = _ZN9wikipedia7article8print_toERSo */

                                                     class wikilink 
                                                           {
                                                                     public:
                                                                              wikilink (std::string const& name);
                                                                                          /* = _ZN9wikipedia7article8wikilinkC1ERKSs */
                                                                                                };
                                                                                                   };
}
```

- ����c�������ӵ�c++
The job of the common C++ idiom:
```cpp
#ifdef __cplusplus 
extern "C" {
#endif
        /* ... */
#ifdef __cplusplus
}
#endif
```

c���Զ�����unmangling��c++����ʱ��Ҫ��������mangling

����:
```cpp
#ifdef __cplusplus
extern "C" {
#endif

    void *memset (void *, int, size_t);
    char *strcat (char *, const char *);
    int   strcmp (const char *, const char *);
    char *strcpy (char *, const char *);

#ifdef __cplusplus
}
#endif

if (strcmp(argv[1], "-x") == 0) 
        strcpy(a, argv[2]);
        else 
                memset (a, 0, sizeof(a));
                ```

                uses the correct, unmangled strcmp and memset. If the extern had not been used, the (SunPro) C++ compiler would produce code equivalent to:
                ```cpp
                if (__1cGstrcmp6Fpkc1_i_(argv[1], "-x") == 0) 
                        __1cGstrcpy6Fpcpkc_0_(a, argv[2]);
                        else 
                                __1cGmemset6FpviI_0_ (a, 0, sizeof(a));
                                ```
