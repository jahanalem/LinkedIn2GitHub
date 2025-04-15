# Discount System Algorithm Documentation

This document explains the detailed algorithm for processing product updates with discount logic in the LiliShop project. The goal is to ensure that product updates (including photo processing, characteristic synchronization, discount validation, and price adjustments) are handled in a robust, maintainable manner.

## Table of Contents

- [Discount System Algorithm Documentation](#discount-system-algorithm-documentation)
  * [Overview](#overview)
  * [System Components](#system-components)
  * [Algorithm Workflow](#algorithm-workflow)
    + [Product Retrieval](#product-retrieval)
    + [Validation of the Update Request](#2-validation-of-the-update-request)
    + [Photo Processing](#photo-processing)
    + [Mapping Update Data](#mapping-update-data)
    + [Discount Handling](#discount-handling)
      - [Discount Validation](#discount-validation)
      - [Discount Update or Creation](#discount-update-or-creation)
      - [Price Changes and Scheduled Discounts](#price-changes-and-scheduled-discounts)
    + [Persistence and Final Response](#persistence-and-final-response)
  * [Discount Scheduling and Lifecycle](#discount-scheduling-and-lifecycle)
  * [Conclusion](#conclusion)
  * [Related Components](#related-components)
  * [🧠 Full Source Code (Backend & Frontend)](#-full-source-code-backend--frontend)
    + [🔧 `UpdateProductAsync` Method — ProductService (Backend)](#-updateproductasync-method--productservice-backend)
    + [🧩 `MapUpdateDtoToProduct` Method — ProductMapper (Backend)](#-mapupdatedtotoproduct--method---productmapper--backend)
      - [✅ Key Responsibilities:](#--key-responsibilities-)
    + [🚀 `submit()` Method — EditProductComponent (Frontend)](#-submit-method--editproductcomponent-frontend)
      - [✅ Key Responsibilities:](#-key-responsibilities-1)
  * [🧩 Discount System Database Schema](#-discount-system-database-schema)
  * [🖼️ UI Preview: Discount Management](#%EF%B8%8F-ui-preview-discount-management)
    + [🎛️ Admin Panel: Discount Form Fields](#%EF%B8%8F-admin-panel-discount-form-fields)
    + [🧾 Customer View: Product Detail Page](#-customer-view-product-detail-page)
    + [🛒 Customer View: Product List Grid](#-customer-view-product-list-grid)
  * [📌 Tags](#-tags)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>


## Overview

The discount system in LiliShop is a complex set of business rules that manage how discounts are applied to products. A discount can be active immediately or scheduled for a future activation. 
Only one discount per product is active at a time. The system handles:

- **Validation:** Ensuring the discount dates and values are correct.
- **Mapping:** Converting incoming DTOs (data transfer objects) into the product entity.
- **Price Adjustment:** Calculating the discounted price while preserving the original price.
- **Discount Lifecycle:** Scheduling activation and deactivation of discounts using background jobs.

---

## System Components

- **`ProductService.UpdateProductAsync`**  
  The main service method that coordinates the update process. It retrieves the product, validates incoming data, maps updates, processes discounts, and persists changes.

- **`ProductMapper.MapUpdateDtoToProduct`**  
  A custom mapper that updates product properties, handles photos and characteristics, and processes discount logic.

- **Validation Methods**  
  Methods like `ValidateProductForUpdate` ensure that discount dates are valid and that business rules (such as not moving an active discount's start date to the future) are enforced.

- **Discount Lifecycle Management**  
  The system schedules background jobs (using Hangfire) for activating future discounts and deactivating expired discounts.

---

## Algorithm Workflow

### Product Retrieval

- The product is fetched from the database in a single query along with all its related entities (photos, characteristics, and discount history).

```csharp
// Fetch product with all related entities in a single call
var productFromDb = await FetchProductFromDatabaseAsync(productId);
```

### Validation of the Update Request

- **Validate Existence**  
  The system checks if the product exists.

- **Discount Date Validation**  
  If a discount is provided in the update DTO, it validates:
  - The discount's end date must be later than the current time.
  - The discount's start date must be before the end date.

- **Active Discount Date Reassignment**  
  If there is an active discount and the new discount’s start date is in the future, the update is rejected. The user is prompted to first deactivate the current discount.

```csharp
var (isValid, validationResult) = ValidateProductForUpdate(productFromDb, productToUpdateDto);
if (!isValid)
{
    return validationResult;
}
```

### Photo Processing

- Any new product photo (if provided) is processed asynchronously. This ensures that updated images are uploaded and associated with the product entity.

```csharp
await ProcessPhotosAsync(productFromDb, file);
```

### Mapping Update Data

The `ProductMapper.MapUpdateDtoToProduct` method maps the update DTO to the existing product entity. This includes:

- **Basic Fields**:  
  Maps general product properties such as:
  - `Name`
  - `Description`
  - `Price`
  - `StockQuantity`
  - `Brand`, `Type`, and other core fields

- **Photos**:  
  Compares the existing `ProductPhoto` list with the `ProductPhotoDto` list:
  - Removes photos not in the new list
  - Updates existing ones
  - Adds new photos with correct ordering and base64 validation

- **Characteristics**:  
  Handles product characteristics by:
  - Removing outdated ones
  - Updating quantities of existing ones based on `SizeClassificationId`
  - Adding new ones as needed

- **Discount Mapping**:
  If a `DiscountDto` is provided:
  - Checks if the product already has a related `ProductDiscount`
  - If not:
    - Creates a new `ProductDiscount`
    - Links it to a new or reused `Discount` entity
  - If yes:
    - Updates the existing `Discount` values if they differ
    - Deactivates the current discount if the update invalidates it
  - Handles `ScheduledPrice`:
    - Creates or updates the associated `ScheduledPrice` entity if `StartDate` or `EndDate` is specified
  - Automatically deactivates any active discount if the new `DiscountDto` is `null`

- **Price Adjustment**:
  - If a discount is active, calculates and sets the correct discounted price directly
  - If a scheduled price is set (future discount), the base price remains until the scheduled job applies the discount
  - The `ProductMapper` manually updates the `CurrentPrice` field according to the discount state and timing

- **Consistency & Validity**:
  - The mapping ensures all entities (`Product`, `ProductDiscount`, `Discount`, `ScheduledPrice`) are properly linked and valid
  - Logic avoids duplicate discounts and ensures only one active discount per product at any time
  - Validation ensures the final `CurrentPrice` is correct and in sync with discount logic


```csharp
_productMapper.MapUpdateDtoToProduct(productToUpdateDto, productFromDb);
```

### Discount Handling

#### Discount Validation

- The system validates the discount data using `ValidateProductForUpdate`.
- A discount change is considered valid if:
  - A discount is provided.
  - The discount is active (either explicitly or implicitly).
  - The new price is lower than the reference (base) price.

#### Discount Update or Creation

- **If the product price has changed**  
  A new discount is created and the previous active discount is replaced.

- **If the product price is unchanged**  
  The existing discount is updated with the new properties.

```csharp
if (dto.Discount is not null)
{
    if (IsValidDiscountUpdate(dto, productFromDb))
    {
        if (existingProduct.Price != dto.Price)
        {
            // Create a new discount (with updated amounts, dates, etc.)
        }
        else
        {
            // Update properties of the existing discount.
        }
    }
    else
    {
        // Mark the discount as inactive.
        GetLatestProductDiscount(existingProduct).Discount.IsActive = false;
    }

    // Apply price changes based on discount status
    ApplyPriceChanges(dto, productFromDb);
}
```

#### Price Changes and Scheduled Discounts

- The method `ApplyPriceChanges` computes the discounted price.
- If the discount is scheduled for the future (its start date is greater than the current time):
  - For future discounts, the new price is scheduled without changing the current price.
- For immediate discounts:
  - The product's price is updated directly, and the old price is stored as `PreviousPrice`.

### Persistence and Final Response

- The updated product entity is saved to the database via the Unit of Work pattern.
- Finally, the product entity is mapped to a return DTO and sent back as a successful operation result.

```csharp
_unitOfWork.Repository<Product>().Update(productFromDb);
await _unitOfWork.CompleteAsync();

var productToReturnDto = _mapper.Map<ProductToReturnDto>(productFromDb);
return new SuccessOperationResult<ProductToReturnDto>(productToReturnDto);
```

---

## Discount Scheduling and Lifecycle

Discount lifecycle events are managed via background jobs:

- **Activation**  
  If a discount is scheduled for a future start date, a job is scheduled (e.g., via Hangfire) to activate the discount at the designated time.

- **Deactivation**  
  At the discount's end date, another job is scheduled to automatically revert prices and deactivate the discount.

- The methods `ScheduleDiscountStartJob` and `ScheduleDiscountEndJobIfNeeded` encapsulate this behavior,
  while methods like `ActivateScheduledDiscountAsync` and `DeactivateDiscountAsync` perform the updates when the jobs execute.

---

## Conclusion

The discount system algorithm is a comprehensive solution that handles complex discount scenarios, including:

1. **Loading products** with their full set of related data (photos, characteristics, discounts) in a single call.
2. **Validating** incoming product update and discount data against strict business rules.
3. **Mapping** update DTOs to product entities via a custom `ProductMapper`, ensuring all related data is properly synchronized.
4. **Processing discount logic** to determine whether to create a new discount, update the current one, or deactivate an invalid discount.
5. **Handling price changes** correctly for both immediate and future discounts, with proper scheduling for activation and deactivation tasks.
6. **Persisting changes** and returning a clean response to the caller.

This architecture ensures that discount changes are applied reliably and that both immediate and future discount events are managed appropriately, with clear validation and error handling.

---

## Related Components

- **ProductService**  
  Orchestrates the update process, calls validation, mapping, discount processing, and persistence methods.

- **ProductMapper**  
  Responsible for mapping ProductToUpdateDto to the Product entity, handling photo and characteristic synchronization, as well as discount logic.

- **UnitOfWork & Repository**  
  Provide data persistence and atomic transaction handling.

- **Discount Scheduling Methods (e.g., ActivateScheduledDiscountAsync, DeactivateDiscountAsync)**  
  Manage timed discount activations/deactivations using background job scheduling.

---

---

## 🧠 Full Source Code (Backend & Frontend)

Below are the core implementations that power the discount update mechanism. These include:

- The **backend service method** that handles product updates
- The **mapper logic** for converting DTOs to domain entities
- The **frontend submission logic** that triggers the update process

---

### 🔧 `UpdateProductAsync` Method — ProductService (Backend)

This method is the main entry point for updating a product. It handles:

- Validating the incoming `ProductToUpdateDto`
- Updating product fields (name, brand, type, etc.)
- Processing product characteristics
- Applying or scheduling discounts
- Managing associated photos
- Triggering background jobs (e.g., for scheduled price changes)

<details>
<summary>Click to expand</summary>
<br>

```csharp
public virtual async Task<IOperationResult<ProductToReturnDto>> UpdateProductAsync(int productId, ProductToUpdateDto productToUpdateDto, IFormFile file = null)
{
    if (productToUpdateDto is null)
    {
        return new FailureOperationResult<ProductToReturnDto>(ErrorCode.InvalidArgument, "The product cannot be null.");
    }

    try
    {
        // Fetch product with all related entities in a single call
        var productFromDb = await FetchProductFromDatabaseAsync(productId);

        var (isValid, validationResult) = ValidateProductForUpdate(productFromDb, productToUpdateDto);
        if (!isValid)
        {
            return validationResult;
        }

        await ProcessPhotosAsync(productFromDb, file);

        _productMapper.MapUpdateDtoToProduct(productToUpdateDto, productFromDb);

        ProcessDiscountLogic(productFromDb, productToUpdateDto?.Discount?.Id);

        // Update product in the database
        _unitOfWork.Repository<Product>().Update(productFromDb);
        await _unitOfWork.CompleteAsync();

        var productToReturnDto = _mapper.Map<ProductToReturnDto>(productFromDb);

        return new SuccessOperationResult<ProductToReturnDto>(productToReturnDto);
    }
    catch (Exception exception)
    {
        _logger.LogError($"An error occurred: {exception.Message}");
        return new FailureOperationResult<ProductToReturnDto>(ErrorCode.GeneralException, "An unexpected error occurred.");
    }
}

private (bool IsValid, IOperationResult<ProductToReturnDto> Result) ValidateProductForUpdate(Product product, ProductToUpdateDto dto)
{
    if (product is null)
    {
        return (false, new FailureOperationResult<ProductToReturnDto>(ErrorCode.ResourceNotFound, "Product not found."));
    }

    Discount currentDiscount = null;
    if (dto?.Discount is not null)
    {
        currentDiscount = product.ProductDiscounts.FirstOrDefault(pd => pd.Discount.Id == dto?.Discount.Id)?.Discount;
    }

    if (currentDiscount is not null)
    {
        if (dto.Discount.EndDate < DateTimeOffset.Now)
        {
            return (false, new FailureOperationResult<ProductToReturnDto>(ErrorCode.InvalidData, "The end date should be less than current time."));
        }
        if (dto.Discount.StartDate > dto.Discount.EndDate)
        {
            return (false, new FailureOperationResult<ProductToReturnDto>(ErrorCode.InvalidData, "The start date should be less than end date."));
        }
    }

    var dtoDiscount = dto.Discount;

    // Validation: User tries to move an active discount's StartDate to the future
    if (currentDiscount != null && dtoDiscount != null && dtoDiscount.StartDate > DateTime.UtcNow)
    {
        return (false, new FailureOperationResult<ProductToReturnDto>(ErrorCode.InvalidData, "You cannot move the StartDate of an active discount to the future. Please deactivate the current discount first, then create a new one."));
    }

    return (true, new SuccessOperationResult<ProductToReturnDto>(null, "The objects are valid."));
}

private async Task<Product> FetchProductFromDatabaseAsync(int productId)
{
    try
    {
        return await _unitOfWork.Repository<Product>()
            .GetByCriteria(p => p.Id == productId)
            .Include(p => p.ProductCharacteristics)
            .Include(p => p.ProductPhotos)
            .Include(p => p.ProductDiscounts).ThenInclude(pd => pd.Discount).AsTracking()
            .FirstOrDefaultAsync();
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, $"Error fetching product with ID {productId}", productId);
        throw;
    }
}

private async Task ProcessPhotosAsync(Product productFromDb, IFormFile file)
{
    await EnsureMainProductPhotoAsync(productFromDb);

    if (file is not null)
    {
        bool photoAdded = await AddProductPhotoFromFileAsync(productFromDb, file);
        if (!photoAdded)
        {
            throw new InvalidOperationException("Failed to add product photo.");
        }
    }
}

private ProductDiscount GetActiveProductDiscount(Product product)
{
    return product.ProductDiscounts.FirstOrDefault(pd => pd.Discount.IsActive);
}
private void ProcessDiscountLogic(Product productFromDb, int? discountId)
{
    if (discountId is null)
    {
        return;
    }
    var activeProductDiscount = GetActiveProductDiscount(productFromDb);
    if (activeProductDiscount is null)
    {
        HandleInactiveDiscount(productFromDb, discountId.Value);
        return;
    }
    try
    {
        HandleActiveDiscount(productFromDb, activeProductDiscount);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error processing discount logic for product {ProductId}", productFromDb.Id);
        throw;
    }
}
private void HandleInactiveDiscount(Product product, int discountId)
{
    RemoveExistingDiscountJobs(product, discountId);
}
private void RemoveExistingDiscountJobs(Product product, int? discountId)
{
    var discountsToProcess = discountId.HasValue
        ? product.ProductDiscounts.Where(pd => pd.DiscountId == discountId.Value && pd.ProductId == product.Id).Take(1)
        : product.ProductDiscounts;

    foreach (var pd in discountsToProcess)
    {
        if (pd.Discount is null)
        {
            continue;
        }

        RemoveJobIfExists(pd.Discount.StartJobId);
        RemoveJobIfExists(pd.Discount.EndJobId);

        pd.Discount.StartJobId = null;
        pd.Discount.EndJobId = null;
    }
}
private void RemoveJobIfExists(string jobId)
{
    if (!string.IsNullOrEmpty(jobId))
    {
        BackgroundJob.Delete(jobId);
    }
}
private void HandleActiveDiscount(Product product, ProductDiscount activePd)
{
    var unsubscribeUrl = _urlHelperService.GenerateUnsubscribeLink();
    var isFutureDiscount = activePd.Discount.StartDate > DateTimeOffset.UtcNow;

    // If user changed DiscountStartDate or DiscountEndDate, remove old jobs
    RemoveExistingDiscountJobs(product, null);

    if (isFutureDiscount)
    {
        HandleFutureDiscount(activePd, unsubscribeUrl);
    }
    else
    {
        ApplyImmediateDiscount(activePd, unsubscribeUrl);
    }

    ScheduleDiscountEndJobIfNeeded(product, activePd);
}
private void HandleFutureDiscount(ProductDiscount activePd, string unsubscribeUrl)
{
    ScheduleDiscountStartJob(activePd, unsubscribeUrl);
}
private void ScheduleDiscountStartJob(ProductDiscount pd, string unsubscribeUrl)
{
    if (!pd.Discount.StartDate.HasValue)
    {
        return;
    }

    var delay = GetValidDelay(pd.Discount.StartDate.Value);
    var jobId = BackgroundJob.Schedule<ProductService>(s => s.ActivateScheduledDiscountAsync(pd.ProductId, unsubscribeUrl), delay);

    pd.Discount.StartJobId = jobId;
}
private TimeSpan GetValidDelay(DateTimeOffset targetDate)
{
    var delay = targetDate - DateTimeOffset.UtcNow;
    return delay > TimeSpan.Zero ? delay : TimeSpan.Zero;
}
private void ApplyImmediateDiscount(ProductDiscount activePd, string unsubscribeUrl)
{
    ScheduleDiscountStartJob(activePd, unsubscribeUrl);
}
private void ScheduleDiscountEndJobIfNeeded(Product product, ProductDiscount activePd)
{
    if (!ShouldScheduleEndJob(activePd))
    {
        return;
    }

    var delay = GetValidDelay(activePd.Discount.EndDate.Value);
    var jobId = BackgroundJob.Schedule<ProductService>(s => s.DeactivateDiscountAsync(product.Id), delay);

    activePd.Discount.EndJobId = jobId;
}
private bool ShouldScheduleEndJob(ProductDiscount activePd)
{
    return activePd.Discount.EndDate.HasValue &&
        activePd.Discount.EndDate > DateTimeOffset.UtcNow &&
        activePd.Discount.IsActive;
}

public async Task ActivateScheduledDiscountAsync(int productId, string unsubscribeBaseUrl)
{
    var product = await _unitOfWork.Repository<Product>()
           .GetByCriteria(p => p.Id == productId)
           .Include(p => p.ProductDiscounts).ThenInclude(pd => pd.Discount).AsTracking()
           .FirstOrDefaultAsync();

    if (product is null)
    {
        return;
    }

    var activePd = product.ProductDiscounts.LastOrDefault(pd => pd.ProductId == product.Id);
    if (activePd is null)
    {
        return;
    }
    _unitOfWork.MarkAsModified<ProductDiscount>(product.ProductDiscounts.FirstOrDefault(pd => pd.DiscountId == activePd.DiscountId));
    // If we have a scheduled price, move it into Price and clear out scheduled price
    if (activePd.ScheduledPrice.HasValue)
    {
        // Keep the old price in PreviousPrice
        product.PreviousPrice = product.Price;
        product.Price = activePd.ScheduledPrice.Value;
        product.ProductDiscounts.FirstOrDefault(pd => pd.DiscountId == activePd.DiscountId).ScheduledPrice = null;
    }

    // Clear the discount start job ID since it's completed
    product.ProductDiscounts.FirstOrDefault(pd => pd.DiscountId == activePd.DiscountId).Discount.StartJobId = null;

    try
    {
        // Save changes
        _unitOfWork.Repository<Product>().Update(product);
        await _unitOfWork.CompleteAsync();
        await _notificationService.NotifyPriceDropAsync(product.Id, unsubscribeBaseUrl);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to activate scheduled discount for Product ID {productId}.", productId);
    }
}
public async Task DeactivateDiscountAsync(int productId)
{
    var product = await _unitOfWork.Repository<Product>()
          .GetByCriteria(p => p.Id == productId)
          .Include(p => p.ProductDiscounts).ThenInclude(pd => pd.Discount).AsTracking()
          .FirstOrDefaultAsync();

    if (product is null)
    {
        _logger.LogWarning($"Product with ID {productId} not found.", productId);
        return;
    }

    // Only change if still active
    var activePd = product.ProductDiscounts.LastOrDefault(pd => pd.ProductId == product.Id);
    if (activePd is not null && activePd.Discount.IsActive)
    {
        activePd.Discount.IsActive = false;
        activePd.Discount.StartDate = null;
        activePd.Discount.EndDate = null;

        product.Price = product.PreviousPrice.Value;
        product.PreviousPrice = null;

        // Remove any leftover job IDs if you stored them
        activePd.Discount.StartJobId = null;
        activePd.Discount.EndJobId = null;

        _unitOfWork.Repository<Product>().Update(product);
        await _unitOfWork.CompleteAsync();

        _logger.LogInformation("Discount automatically deactivated for Product ID {productId}.", productId);
    }
}

```
</details>

---

### 🧩 `MapUpdateDtoToProduct` Method — ProductMapper (Backend)

This method is responsible for transforming a `ProductToUpdateDto` into an existing `Product` entity. 
It ensures that all product-related information is accurately updated, including core fields, discount logic, characteristics, and photos.

#### ✅ Key Responsibilities:

- Map basic product fields (e.g., `Name`, `Brand`, `Type`, `Price`, `Description`)
- Apply discount updates based on the DTO:
  - If a new discount is provided, create and attach it
  - If an existing discount needs an update, update it
  - If no discount is provided but one exists, deactivate the existing one
  - If a scheduled price is provided, validate it and schedule accordingly
- Calculate and apply discounted or scheduled price
- Map product characteristics:
  - Add new ones
  - Update quantities for existing ones
  - Remove any that are no longer present
- Sync photo collection with the current state in the DTO:
  - Add new photos
  - Remove deleted ones

<details>
<summary>Click to expand</summary>
<br>

```csharp
namespace Lili.Shop.Service.Mappers
{
    public class ProductMapper
    {
        private readonly IUnitOfWork _unitOfWork;
        private readonly IConfiguration _config;
        public ProductMapper(IUnitOfWork unitOfWork, IConfiguration config)
        {
            _unitOfWork = unitOfWork;
            _config = config;
        }

        /// <summary>
        /// Maps the update DTO to an existing product entity.
        /// This includes updating basic properties, photos, characteristics, and discount logic.
        /// Scheduled discount jobs, StartJobId and EndJobId, are handled in:
        /// <see cref="ProductService.ActivateScheduledDiscountAsync"/> and 
        /// <see cref="ProductService.DeactivateDiscountAsync"/>. 
        /// </summary>
        /// <param name="dto">The product DTO containing updated values.</param>
        /// <param name="existingProduct">The existing product entity from the database.</param>
        /// <returns>The updated product entity.</returns>
        public Product MapUpdateDtoToProduct(ProductToUpdateDto dto, Product existingProduct)
        {
            existingProduct.Name = dto.Name;
            existingProduct.Description = dto.Description;
            existingProduct.ProductTypeId = dto.ProductTypeId;
            existingProduct.ProductBrandId = dto.ProductBrandId;
            existingProduct.IsActive = dto.IsActive;
            existingProduct.PictureUrl = HandlePictureUrl(dto, existingProduct);
            existingProduct.ProductPhotos = HandleProductPhotos(dto, existingProduct);

            if (dto.ProductCharacteristics is not null)
            {
                HandleProductCharacteristics(existingProduct, dto.ProductCharacteristics);
            }

            if (dto.Discount is not null)
            {
                // It means there is a discount and one or some of its props was changed.
                if (IsValidDiscountUpdate(dto, existingProduct))
                {
                    if (existingProduct.Price != dto.Price)
                    {
                        // if the price is changed, we need to create a new discount and make the last discount to deactivate
                        var productDiscount = new ProductDiscount
                        {
                            ProductId = existingProduct.Id,
                            Discount = new Discount
                            {
                                Name = string.IsNullOrWhiteSpace(dto.Discount.Name) ? "Single Discount" : dto.Discount.Name,
                                Amount = existingProduct.Price - dto.Price,
                                IsPercentage = dto.Discount.IsPercentage.GetValueOrDefault(),
                                IsActive = dto.Discount.IsActive.GetValueOrDefault(),
                                StartDate = dto.Discount.StartDate,
                                EndDate = dto.Discount.EndDate,
                                IsFreeShipping = dto.Discount.IsFreeShipping.GetValueOrDefault(),
                            }
                        };

                        existingProduct.ProductDiscounts = new List<ProductDiscount> { productDiscount };
                    }
                    else
                    {
                        // if the price is not changed, we need to update the existing discount
                        var existingDiscount = GetLatestProductDiscount(existingProduct)?.Discount;
                        if (existingDiscount is not null)
                        {
                            UpdateDiscountProperties(existingDiscount, dto, existingProduct);
                        }
                    }
                }
                else
                {
                    if (existingProduct.ProductDiscounts is not null)
                    {
                        GetLatestProductDiscount(existingProduct).Discount.IsActive = false;
                    }
                }

                ApplyPriceChanges(dto, existingProduct);
            }

            return existingProduct;
        }

        private void UpdateDiscountProperties(Discount existingDiscount, ProductToUpdateDto dto, Product existingProduct)
        {
            if (dto.Discount.IsActive.HasValue && existingDiscount.IsActive != dto.Discount.IsActive.Value)
            {
                existingDiscount.IsActive = dto.Discount.IsActive.Value;
            }

            if (!string.IsNullOrEmpty(dto.Discount.Name) && existingDiscount.Name != dto.Discount.Name)
            {
                existingDiscount.Name = dto.Discount.Name;
            }

            var newAmount = UpdateDiscountAmount(dto, existingProduct);
            if (dto.Discount.Amount != default && existingDiscount.Amount != newAmount)
            {
                existingDiscount.Amount = newAmount;
            }

            if (dto.Discount.IsPercentage.HasValue && existingDiscount.IsPercentage != dto.Discount.IsPercentage.Value)
            {
                existingDiscount.IsPercentage = dto.Discount.IsPercentage.Value;
            }

            if (dto.Discount.IsFreeShipping.HasValue && existingDiscount.IsFreeShipping != dto.Discount.IsFreeShipping.Value)
            {
                existingDiscount.IsFreeShipping = dto.Discount.IsFreeShipping.Value;
            }

            if (dto.Discount.StartDate.HasValue && existingDiscount.StartDate != dto.Discount.StartDate)
            {
                existingDiscount.StartDate = dto.Discount.StartDate;
            }

            if (dto.Discount.EndDate.HasValue && existingDiscount.EndDate != dto.Discount.EndDate)
            {
                existingDiscount.EndDate = dto.Discount.EndDate;
            }
        }

        private decimal UpdateDiscountAmount(ProductToUpdateDto dto, Product existingProduct)
        {
            if (!dto.Discount.IsActive.GetValueOrDefault())
            {
                var prodDiscount = existingProduct.ProductDiscounts.FirstOrDefault(pd => pd.Discount.IsActive && pd.DiscountId == dto.Discount.Id);
                return prodDiscount?.Discount?.Amount ?? 0;
            }

            if (existingProduct.PreviousPrice.HasValue)
            {
                return existingProduct.PreviousPrice.Value - dto.Price;
            }

            return existingProduct.Price - dto.Price;
        }

        private void ApplyPriceChanges(ProductToUpdateDto dto, Product existingProduct)
        {
            if (dto.Discount is null)
            {
                existingProduct.Price = dto.Price;
                return;
            }
            if (dto.Discount != null)
            {
                DateTimeOffset? startDate = null;
                if (dto.Discount.IsActive.GetValueOrDefault() == true)
                {
                    startDate = dto.Discount.StartDate.Value;
                }
                else
                {
                    startDate = GetLatestProductDiscount(existingProduct)?.Discount.StartDate;
                }

                DateTimeOffset? endDate = null;
                if (dto.Discount.IsActive.GetValueOrDefault() == true)
                {
                    endDate = dto.Discount.EndDate.Value;
                }
                else
                {
                    endDate = GetLatestProductDiscount(existingProduct)?.Discount.EndDate;
                }

                bool isFutureDiscount = startDate.HasValue && startDate > DateTimeOffset.UtcNow;

                var discountedPrice = CalculateDiscountedPrice(dto.Price, dto.Discount.Amount.GetValueOrDefault(), dto.Discount.IsPercentage.GetValueOrDefault());
                // The user want to deactivate the existing discount
                if (!IsDiscountCurrentlyActive(dto))
                {
                    if (isFutureDiscount)
                    {
                        existingProduct.Price = discountedPrice;
                        GetLatestProductDiscount(existingProduct).ScheduledPrice = null;
                        // we expect to PreviousPrice is null
                    }
                    else
                    {
                        existingProduct.Price = existingProduct.PreviousPrice.GetValueOrDefault();
                        existingProduct.PreviousPrice = null;
                        // we expect to ScheduledPrice is null
                    }
                }
                else
                {
                    if (!IsImplicitlyActiveDiscount(dto))
                    {
                        if (isFutureDiscount)
                        {
                            // existingProduct.Price is not changed.
                            GetLatestProductDiscount(existingProduct).ScheduledPrice = discountedPrice;
                            // we expect to PreviousPrice is null
                        }
                        else
                        {
                            existingProduct.PreviousPrice = existingProduct.Price;
                            existingProduct.Price = discountedPrice;
                            // we expect to ScheduledPrice is null
                        }
                    }
                }
            }
        }

        private decimal CalculateDiscountedPrice(decimal price, decimal discountAmount, bool isPercentage)
        {
            return isPercentage ? price * (1 - (discountAmount / 100m)) : price - discountAmount;
        }

        /// <summary>
        /// Determines if the incoming discount changes should be applied based on activation status and price validity.
        /// A valid discount change requires an active discount (explicit or implied) and a new price lower than the current reference price.
        /// </summary>
        /// <remarks>
        /// Reference price is either the previous undiscounted price (if exists) or the current base price.
        /// </remarks>
        private bool IsValidDiscountUpdate(ProductToUpdateDto dto, Product existingProduct)
        {
            if (dto.Discount is null)
            {
                // It means there is no discount or there is one but no changes have been made.
                return false;
            }
            var referencePrice = (existingProduct.PreviousPrice.HasValue && existingProduct.PreviousPrice.Value > 0)
                ? existingProduct.PreviousPrice.GetValueOrDefault()
                : existingProduct.Price;

            bool isActiveDiscount = IsDiscountCurrentlyActive(dto);

            return (dto.Discount != null && isActiveDiscount && dto.Price < referencePrice);
        }

        /// <summary>
        /// Determines if the discount should be considered active based on explicit flag or implicit activation
        /// </summary>
        /// <remarks>
        /// Implicit activation occurs when discount properties are modified without explicitly setting IsActive
        /// </remarks>
        private bool IsDiscountCurrentlyActive(ProductToUpdateDto dto)
        {
            bool isActive = false;
            if (IsImplicitlyActiveDiscount(dto))
            {
                isActive = true;
            }
            else
            {
                isActive = dto.Discount.IsActive.GetValueOrDefault();
            }

            return isActive;
        }

        /// <summary>
        /// Determines if the discount is implicitly active (IsActive not specified but discount properties are present)
        /// </summary>
        /// <remarks>
        /// Used when discount modifications don't include an explicit IsActive flag,
        /// indicating the discount should maintain/update its current active state
        /// </remarks>
        private bool IsImplicitlyActiveDiscount(ProductToUpdateDto dto)
        {
            return (dto.Discount != null && !dto.Discount.IsActive.HasValue);
        }

        private ProductDiscount? GetLatestProductDiscount(Product product)
        {
            return product.ProductDiscounts.LastOrDefault();
        }

        private void HandleProductCharacteristics(Product productFromDb, List<ProductCharacteristicDto> newItemsDto)
        {
            // old list from DB
            var oldList = productFromDb.ProductCharacteristics;

            // Convert the incoming Dto to real ProductCharacteristic entities:
            // If Id>0 => existing. If Id=0 => new item.
            var newList = newItemsDto?.Select(x => new ProductCharacteristic
            {
                Id = x.Id,  // If it's an existing row, should be >0. 
                ProductId = productFromDb.Id,
                SizeClassificationId = x.SizeId,
                Quantity = x.Quantity
            }).ToList() ?? new();

            // 1) Delete old items that are not in the new request
            //    Matching on SizeClassificationId or Id, whichever is your unique
            var toRemove = oldList
                .Where(dbItem => !newList.Any(req => req.SizeClassificationId == dbItem.SizeClassificationId))
                .ToList();

            foreach (var oldItem in toRemove)
            {
                // oldItem must have Id>0 if it's truly from DB
                _unitOfWork.Repository<ProductCharacteristic>().Delete(oldItem);
            }

            // 2) For each new item
            //    - if it matches existing by SizeId => update
            //    - else add
            foreach (var newItem in newList)
            {
                var dbItem = oldList.FirstOrDefault(db => db.SizeClassificationId == newItem.SizeClassificationId);

                if (dbItem is null)
                {
                    // It's truly new
                    _unitOfWork.Repository<ProductCharacteristic>().Add(newItem);
                }
                else
                {
                    // It's existing => update quantity
                    dbItem.Quantity = newItem.Quantity;
                }
            }
        }

        private List<ProductPhoto> HandleProductPhotos(ProductToUpdateDto dto, Product existingProduct)
        {
            if (dto?.ProductPhotos is null)
            {
                return existingProduct.ProductPhotos.ToList();
            }

            var result = dto.ProductPhotos.Select(photo => new ProductPhoto
            {
                Url = photo.Url,
                IsMain = photo.IsMain,
                PublicId = photo.PublicId,
                ProductId = existingProduct.Id,
                Product = existingProduct,
            }).ToList();

            return result;
        }

        private string HandlePictureUrl(ProductToUpdateDto dto, Product existingProduct)
        {
            return !string.IsNullOrWhiteSpace(dto.PictureUrl)
                     ? dto.PictureUrl.Replace(_config["ApiUrl"] ?? string.Empty, "")
                     : string.Empty;
        }
    }
}

```
</details>

---

### 🚀 `submit()` Method — EditProductComponent (Frontend)

This method is triggered when the product update form is submitted. It performs client-side validations, constructs the update DTO including discount and other fields, and calls the backend API to update the product.

#### ✅ Key Responsibilities:

- Validate the form before proceeding
- Extract values from the form and map them to `ProductToUpdateDto`
- Format and attach discount information (including start/end dates)
- Handle uploaded photos and characteristics
- Call the backend service method `updateProduct`
- Handle success and error responses from the API

<details>
<summary>Click to expand</summary>
<br>

```typescript
onSubmit() {
    if (this.productForm.invalid) {
      this.toastr.error('Please fill out the form correctly.', 'Form Validation Error');
      return;
    }

    const formValues      = this.productForm.value;
    const existingProduct = this.product();
    const isUpdate        = !!existingProduct && existingProduct.id > 0;

    // Combine date and time for discount start and end

    if (formValues.isDiscountActive) {
      if (!formValues.discountStartDateDate) {
        formValues.discountStartDateDate = new Date();
      }
      this.backupStartTime = formValues.discountStartDateTime;
      this.backupEndTime   = formValues.discountEndDateTime;
    }

    // Construct the payload
    let discount: Partial<IDiscount> | null = null;

    const oldDiscount = this.product()?.discount;
    const hasExistingDiscount = !!oldDiscount;

    if (formValues.isDiscountActive !== null && formValues.isDiscountActive !== undefined) {
      const updatedDiscount: Partial<IDiscount> = {};
      let hasChanges = false;

      if (hasExistingDiscount) {
        updatedDiscount.id = oldDiscount.id;
      }

      if (!hasExistingDiscount || oldDiscount.isActive !== formValues.isDiscountActive) {
        updatedDiscount.isActive = formValues.isDiscountActive;
        hasChanges = true;
      }

      const defaultName = "Single Discount";
      if (!hasExistingDiscount || oldDiscount.name !== defaultName) {
        updatedDiscount.name = defaultName;
        hasChanges = true;
      }

      if (!hasExistingDiscount || oldDiscount.isPercentage !== false) {
        updatedDiscount.isPercentage = false;
        hasChanges = true;
      }

      if (formValues.discountStartDateDate) {
        const newStart = this.combineDateAndTime(formValues.discountStartDateDate, formValues.discountStartDateTime);
        const oldStart = oldDiscount?.startDate ? new Date(oldDiscount.startDate) : null;

        if (!hasExistingDiscount || !oldStart || oldStart.getTime() !== newStart?.getTime()) {
          updatedDiscount.startDate = newStart?.toISOString();
          hasChanges = true;
        }
      }


      if (formValues.discountEndDateDate) {
        const newEnd = this.combineDateAndTime(formValues.discountEndDateDate, formValues.discountEndDateTime);
        const oldEnd = oldDiscount?.endDate ? new Date(oldDiscount.endDate) : null;

        if (!hasExistingDiscount || !oldEnd || oldEnd.getTime() !== newEnd?.getTime()) {
          updatedDiscount.endDate = newEnd?.toISOString();
          hasChanges = true;
        }
      }


      discount = hasChanges ? updatedDiscount : null;
      console.log("updated discount = ", discount);
    } else {
      discount = null;
    }

    const productPayload: Partial<IProduct> = {
      id                    : existingProduct?.id || 0,
      name                  : formValues.name,
      description           : formValues.description,
      price                 : formValues.price,
      previousPrice         : formValues.previousPrice,
      scheduledPrice        : formValues.scheduledPrice,
      pictureUrl            : existingProduct?.pictureUrl || '',
      productType           : existingProduct?.productType,
      productTypeId         : formValues.productTypeId,
      productBrand          : existingProduct?.productBrand,
      productBrandId        : formValues.productBrandId,
      isActive              : formValues.isActive,
      productCharacteristics: formValues.productCharacteristics || [],
      discount              : discount
    };

    // Determine the action: update or create
    const productAction = isUpdate
      ? this.productService.updateProduct(productPayload as IProduct)
      : this.productService.createProduct(productPayload as IProduct);

    // Execute the action and handle the response
    productAction.pipe(takeUntil(this.destroy$)).subscribe({
      next: (updatedProduct) => {
        // Update the product signal with the response
        this.product.set(updatedProduct as IProduct);

        // Patch the form with the updated product data, including productPhotos
        this.productForm.patchValue({
          productPhotos   : updatedProduct.productPhotos,
          previousPrice   : updatedProduct.previousPrice,
          scheduledPrice  : updatedProduct.scheduledPrice,
          price           : updatedProduct.price,
          isDiscountActive: updatedProduct.discount?.isActive,
        });
        // Mark the form as pristine
        this.productForm.valueChanges.pipe(takeUntil(this.destroy$)).subscribe(() => this.cdRef.detectChanges());
        this.productForm.markAsPristine();

        // Show success message
        const action = isUpdate ? 'updated' : 'created';
        this.toastr.success(`Product ${action} successfully`, 'Success');

        // Trigger change detection
        this.cdRef.detectChanges();
      },
      error: (err) => {
        this.toastr.error('An error occurred while saving the product.', 'Error');
        console.error('Error saving product:', err);
      },
    });
  }
```
</details>

---

## 🧩 Discount System Database Schema

The following diagram illustrates the key database tables and their relationships involved in the discount management process:

- **Product**: The main product entity.
- **Discount**: Stores information about discount percentage and duration.
- **ProductDiscount**: A join table that connects products with their assigned discounts and holds scheduled pricing info.
- **State(Hangfire)**: Manages background job scheduling, such as deactivating discounts or applying scheduled prices.

This schema plays a central role in applying, scheduling, and managing product discounts in the LiliShop system.

> 📸 *Diagram below:*

![discount-system_database](https://github.com/user-attachments/assets/35497614-4aa0-49b3-b122-16dd396a1152)

---

## 🖼️ UI Preview: Discount Management

Below are screenshots from the LiliShop UI showcasing how the discount system appears for both admin users and customers.

---

### 🎛️ Admin Panel: Discount Form Fields
> This form allows admins to create or update discounts for a product. It includes fields for the discount price, start and end dates, and scheduled pricing options.

![discount-system_form-fields](https://github.com/user-attachments/assets/b3a283fa-1762-42d9-b6e5-2e7d1d183a17)

---

### 🧾 Customer View: Product Detail Page
> When a product is under discount, users see a **struck-through original price**, the **new discounted price**, and a **live countdown timer** showing how much time is left for the offer.

![discount-system_UI-product](https://github.com/user-attachments/assets/52a849f9-cd9d-430e-9ef3-804b2e25b7e6)

---

### 🛒 Customer View: Product List Grid
> In the product list, discounted products are visually marked with a **label or ribbon** on the card corner, clearly indicating a special offer is active.

![discount-system_UI-products](https://github.com/user-attachments/assets/92cdff2c-0c55-4c32-8581-1e30564f3cbe)


---

## 📌 Tags

#LiliShop &nbsp; #NET &nbsp; #Angular &nbsp; #Discount &nbsp; #Hangfire &nbsp; #BackgroundTask &nbsp; #Csharp &nbsp; #Typescript



