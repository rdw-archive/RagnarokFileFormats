# Unit Shadows

For any given unit, the client renders a shadow effect by adding a special semi-transparent sprite and scaling it to cover an appropriate area of the ground that the unit is perceived to be standing on.

The scaling factor for a given unit is listed in the ``ShadowTable.lub`` file. I assume that any units not listed there will simply default to a factor of ``1.0``.

In older versions of the client, it appears this data was hardcoded and initialized manually (see below)

## Sources

The decompiled version of a 2008 client executable contained this function:

````
/* public: void __thiscall CSession::InitShadowFactorWithJob(void) */
void __thiscall InitShadowFactorWithJob(CSession *this)

{
  vector<float,std::allocator<float>> *this_00;
  float *pfVar1;
  uint uVar2;
  CSession **ppCVar3;
  CSession *local_8;

  this_00 = &this->m_shadowFactorTable;
  local_8 = this;
  erase((vector<float,class_std::allocator<float>_> *)this_00,(this->m_shadowFactorTable)._First,
        (this->m_shadowFactorTable)._Last);
  local_8 = (CSession *)0x3f800000;
  uVar2 = size((vector<float,class_std::allocator<float>_> *)this_00);
  if (uVar2 < 0x61ba) {
    pfVar1 = (this->m_shadowFactorTable)._Last;
    ppCVar3 = &local_8;
    uVar2 = size((vector<float,class_std::allocator<float>_> *)this_00);
    insert((vector<float,class_std::allocator<float>_> *)this_00,pfVar1,0x61ba - uVar2,
           (float *)ppCVar3);
  }
  else {
    uVar2 = size((vector<float,class_std::allocator<float>_> *)this_00);
    if (0x61ba < uVar2) {
      erase((vector<float,class_std::allocator<float>_> *)this_00,
            (this->m_shadowFactorTable)._First + 0x61ba,(this->m_shadowFactorTable)._Last);
    }
  }
  *(this->m_shadowFactorTable)._First = 1.00000000;
  (this->m_shadowFactorTable)._First[1] = 1.00000000;
  (this->m_shadowFactorTable)._First[2] = 1.00000000;
  (this->m_shadowFactorTable)._First[3] = 1.00000000;
  (this->m_shadowFactorTable)._First[4] = 1.00000000;
  (this->m_shadowFactorTable)._First[5] = 1.00000000;
  (this->m_shadowFactorTable)._First[6] = 1.00000000;
	... omitted for brevity ...
  (this->m_shadowFactorTable)._First[0x2f7] = 1.00000000;
  (this->m_shadowFactorTable)._First[0x2f8] = 1.00000000;
  (this->m_shadowFactorTable)._First[0x301] = 1.00000000;
  return;
}
````
