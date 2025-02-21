# **More Speed and Cleaner Code with async/await and Promise.all**

In my last update, I improved the **product loading process** by using **async/await and Promise.all**. This made the code **faster** and **easier to read**.

## **What Was the Problem?**

Before, some important methods were **hidden inside `loadProduct()`**. This made the code **hard to understand**.

### **Old Code (Problem):**
```typescript
loadProduct(): void {
  this.activatedRoute.paramMap.pipe(
    switchMap((params: ParamMap) => {
      const id = params.get('id');
      this.productIdFromUrl = (id === null ? 0 : +id);
      if (this.productIdFromUrl > 0) {
        return this.productService.getProduct(this.productIdFromUrl);
      } else {
        return EMPTY;
      }
    }),
    catchError((error: any) => {
      this.handleApiError(error);
      return EMPTY;
    })
  ).pipe(takeUntil(this.destroy$)).subscribe((prod) => {
    if (prod) {
      this.product.set(prod);
      this.getProductFormValues();   // 🐈 Hidden method calls
      this.productForm.setControl('productCharacteristics', this.formBuilder.array([]));
      this.loadArrayOfDropDownSize();
      this.setProductInLocalStorage();
    }
  });
}
```
### **🚨 The Problem:**
- `getProductFormValues()`, `loadArrayOfDropDownSize()`, and `setProductInLocalStorage()` **were hidden inside `loadProduct()`**.
- You had to **look inside `loadProduct()`** to understand what happens.
- The code was **not clear** and **hard to maintain**.



## **The New Solution: async/await & Promise.all**  

Now, all methods **run directly in `ngOnInit()`**.  

💚 **More clear** – All important steps are inside `ngOnInit()`.  
💚 **Better performance** – `Promise.all()` runs multiple loading tasks **at the same time**.  
💚 **Cleaner code** – `loadProduct()` now **only loads the product**.  

### **🌟 New Code:**
```typescript
async ngOnInit(): Promise<void> {
  try {
    this.createProductForm();

    await Promise.all([
      this.loadBrands(),
      this.loadTypes(),
      this.loadSizes(),
      this.loadProduct()
    ]);

    await Promise.all([
      this.getProductFormValues(),
      this.loadArrayOfDropDownSize(),
      this.setProductInLocalStorage()
    ]);

    await this.updateIsDiscountActiveControl();
    await this.setupDiscountValidation();

  } catch (error) {
    console.error("Error in ngOnInit:", error);
  }
}
```



## **New `loadProduct()` Method**  
The new `loadProduct()` method is **much simpler**. It **only loads the product** and does nothing else.  

### **🌟 New Code for `loadProduct()`:**
```typescript
async loadProduct(): Promise<void> {
  try {
    const prod = await firstValueFrom(
      this.activatedRoute.paramMap.pipe(
        switchMap((params: ParamMap) => {
          const id = params.get('id');
          this.productIdFromUrl = (id === null ? 0 : +id);
          if (this.productIdFromUrl > 0) {
            return this.productService.getProduct(this.productIdFromUrl);
          } else {
            return of(null);
          }
        }),
        catchError((error: any) => {
          this.handleApiError(error);
          return of(null);
        }),
        takeUntil(this.destroy$)
      )
    );

    if (prod) {
      this.product.set(prod);

      this.productForm.setControl('productCharacteristics', this.formBuilder.array([]));
    }
  } catch (error) {
    console.error("Error loading product:", error);
  }
}
```

With these changes, my code is now **cleaner, faster, and easier to maintain**! 🚀✨

https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/shop

https://github.com/jahanalem/LiliShop-frontend-angular

---
