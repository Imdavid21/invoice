<script>
  import { Plus, Trash2, ImagePlus, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    // ... other currencies
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  let loadingLogo = false;

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && ['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) {
      loadingLogo = true;
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        loadingLogo = false;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc.trim() !== '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc.trim(),
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2)
        }
      ];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    } else {
      alert('Please enter valid item details.');
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Your Company Name', logo: '' },
      invoice: {
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: ''
      },
      currency: 'USD'
    };

    save();
  }

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items
    .reduce((total, item) => {
      return total + item.price * item.quantity;
    }, 0)
    .toFixed(2);

  $: discountAmount = ((subTotal * (+appState.discountPercent || 0)) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = ((taxableAmount * (+appState.taxPercent || 0)) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  // Toggles
  let isDiscountEnabled = appState.discountPercent !== '';
  let isTaxEnabled = appState.taxPercent !== '';
</script>

<svelte:body on:click={() => save()} />

<div class="container">
  <!-- Header Section -->
  <header class="header">
    <div class="company-info">
      {#if appState.company.logo}
        <div class="logo-section">
          {#if loadingLogo}
            <div class="loading-spinner">Loading...</div>
          {:else}
            <img src={appState.company.logo} alt="Company Logo" class="company-logo" />
            <button
              class="icon-button"
              on:click={() => {
                appState.company.logo = '';
                save();
              }}
              aria-label="Remove Logo"
            >
              <Trash2 />
            </button>
          {/if}
        </div>
      {:else}
        <label class="logo-upload">
          <ImagePlus />
          <span>Upload Logo</span>
          <input
            type="file"
            accept=".png,.jpg,.jpeg,.webp"
            on:change={handleImageChange}
            class="hidden"
          />
        </label>
      {/if}
      <input
        class="company-name"
        type="text"
        bind:value={appState.company.name}
        placeholder="Your Company Name"
      />
    </div>
    <div class="invoice-info">
      <h1>Invoice</h1>
      <div class="invoice-details">
        <label>
          <span>Invoice #</span>
          <input type="text" bind:value={appState.invoice.number} placeholder="#0001" />
        </label>
        <label>
          <span>Date</span>
          <input type="date" bind:value={appState.invoice.created} />
        </label>
        <label>
          <span>Due Date</span>
          <input type="date" bind:value={appState.invoice.due} />
        </label>
        <label>
          <span>Currency</span>
          <select bind:value={appState.currency}>
            {#each currencyOptions as option}
              <option value={option.code}>
                {option.flag} {option.code} - {option.name}
              </option>
            {/each}
          </select>
        </label>
      </div>
    </div>
  </header>

  <hr />

  <!-- Sender and Recipient Section -->
  <section class="addresses">
    <div class="address-block">
      <h2>From</h2>
      <textarea
        placeholder="Your address"
        bind:value={appState.invoice.from}
        rows="4"
      ></textarea>
      <label>
        <span>{appState.invoice.fromTaxIdName || 'Tax Name'}</span>
        <input type="text" bind:value={appState.invoice.fromTaxId} />
      </label>
      <label>
        <span>Email</span>
        <input type="email" bind:value={appState.invoice.fromContact.mail} />
      </label>
      <label>
        <span>Phone</span>
        <input type="tel" bind:value={appState.invoice.fromContact.phone} />
      </label>
    </div>
    <div class="address-block">
      <h2>To</h2>
      <textarea
        placeholder="Client's address"
        bind:value={appState.invoice.to}
        rows="4"
      ></textarea>
      <label>
        <span>{appState.invoice.toTaxIdName || 'Tax Name'}</span>
        <input type="text" bind:value={appState.invoice.toTaxId} />
      </label>
      <label>
        <span>Email</span>
        <input type="email" bind:value={appState.invoice.toContact.mail} />
      </label>
      <label>
        <span>Phone</span>
        <input type="tel" bind:value={appState.invoice.toContact.phone} />
      </label>
    </div>
  </section>

  <hr />

  <!-- Items Section -->
  <section class="items-section">
    <h2>Invoice Items</h2>
    <form class="item-form" on:submit|preventDefault={addItem}>
      <input
        id="descBox"
        type="text"
        bind:value={itemDesc}
        placeholder="Item Description"
        required
      />
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        required
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        required
      />
      <button type="submit" class="add-button" aria-label="Add Item">
        <Plus />
      </button>
    </form>

    <table class="items-table">
      <thead>
        <tr>
          <th>Description</th>
          <th>Unit Price ({getCurrencySymbol(appState.currency)})</th>
          <th>Quantity</th>
          <th>Total ({getCurrencySymbol(appState.currency)})</th>
          <th class="actions">Actions</th>
        </tr>
      </thead>
      <tbody>
        {#each appState.items as item, index}
          <tr>
            <td>
              <input
                type="text"
                bind:value={item.desc}
                on:input={() => save()}
                required
              />
            </td>
            <td>
              <input
                type="number"
                bind:value={item.price}
                on:input={() => save()}
                min="0"
                step="0.01"
                required
              />
            </td>
            <td>
              <input
                type="number"
                bind:value={item.quantity}
                on:input={() => save()}
                min="1"
                step="1"
                required
              />
            </td>
            <td>{(item.price * item.quantity).toFixed(2)}</td>
            <td class="actions">
              <button
                class="icon-button"
                on:click={() => deleteItem(index)}
                aria-label="Delete Item"
              >
                <Trash2 />
              </button>
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
  </section>

  <!-- Totals Section -->
  <section class="totals">
    <div class="totals-row">
      <span>Subtotal</span>
      <span>{subTotal}</span>
    </div>
    <div class="totals-row">
      <span>
        Discount (%)
        <label class="switch">
          <input type="checkbox" bind:checked={isDiscountEnabled} />
          <span class="slider"></span>
        </label>
      </span>
      <span>
        <input
          type="number"
          bind:value={appState.discountPercent}
          on:input={() => save()}
          min="0"
          max="100"
          step="0.01"
          disabled={!isDiscountEnabled}
        />
      </span>
    </div>
    <div class="totals-row">
      <span>Taxable Amount</span>
      <span>{taxableAmount}</span>
    </div>
    <div class="totals-row">
      <span>
        Tax (%)
        <label class="switch">
          <input type="checkbox" bind:checked={isTaxEnabled} />
          <span class="slider"></span>
        </label>
      </span>
      <span>
        <input
          type="number"
          bind:value={appState.taxPercent}
          on:input={() => save()}
          min="0"
          max="100"
          step="0.01"
          disabled={!isTaxEnabled}
        />
      </span>
    </div>
    <div class="totals-row total-due">
      <span>Total Due</span>
      <span>{totalDue}</span>
    </div>
  </section>

  <!-- Payment Info Section -->
  <section class="payment-info">
    <h2>Payment Information</h2>
    <div class="payment-fields">
      <label>
        <span>Account Number</span>
        <input
          type="text"
          bind:value={appState.payment.accountNumber}
          maxlength="20"
        />
      </label>
      <label>
        <span>Account Name</span>
        <input
          type="text"
          bind:value={appState.payment.accountName}
          maxlength="28"
        />
      </label>
      <label>
        <span>Bank Name</span>
        <input
          type="text"
          bind:value={appState.payment.bank}
          maxlength="28"
        />
      </label>
      <label>
        <span>IFSC</span>
        <input
          type="text"
          bind:value={appState.payment.ifsc}
          maxlength="11"
        />
      </label>
      <label>
        <span>SWIFT Code</span>
        <input
          type="text"
          bind:value={appState.payment.swiftCode}
          maxlength="11"
        />
      </label>
    </div>
  </section>

  <!-- Download Button -->
  <footer class="footer">
    <button class="download-button" on:click={() => window.print()}>
      <Download />
      <span>Download Invoice</span>
    </button>
  </footer>
</div>

<style>
  /* Fonts */
  @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');

  /* Reset and Base Styles */
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  .hidden {
    display: none;
  }

  body {
    font-family: 'Roboto', sans-serif;
    background-color: #f5f7fa;
    color: #333;
    line-height: 1.6;
  }

  /* Container */
  .container {
    max-width: 800px;
    margin: 2rem auto;
    background-color: #fff;
    padding: 2rem;
    border: 1px solid #d1d5db;
    border-radius: 8px;
  }

  /* Header */
  .header {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 2rem;
  }

  .company-info,
  .invoice-info {
    flex: 1 1 45%;
  }

  .logo-section {
    display: flex;
    align-items: center;
  }

  .company-logo {
    max-height: 80px;
    margin-right: 1rem;
  }

  .icon-button {
    background: none;
    border: none;
    cursor: pointer;
    color: #dc2626;
  }

  .icon-button:hover {
    color: #b91c1c;
  }

  .logo-upload {
    display: flex;
    align-items: center;
    cursor: pointer;
    color: #2563eb;
  }

  .logo-upload:hover {
    color: #1d4ed8;
  }

  .company-name {
    font-size: 1.5rem;
    font-weight: 700;
    margin-top: 1rem;
    border: none;
    border-bottom: 1px solid #d1d5db;
    width: 100%;
  }

  .company-name:focus {
    outline: none;
    border-bottom-color: #2563eb;
  }

  .invoice-info h1 {
    font-size: 2rem;
    font-weight: 700;
    margin-bottom: 1rem;
  }

  .invoice-details label {
    display: flex;
    flex-direction: column;
    margin-bottom: 0.5rem;
  }

  .invoice-details span {
    font-size: 0.9rem;
    color: #6b7280;
    margin-bottom: 0.25rem;
  }

  .invoice-details input,
  .invoice-details select {
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
  }

  .invoice-details input:focus,
  .invoice-details select:focus {
    outline: none;
    border-color: #2563eb;
  }

  /* Addresses */
  .addresses {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    margin-bottom: 2rem;
  }

  .address-block {
    flex: 1 1 45%;
    margin-bottom: 1rem;
  }

  .address-block h2 {
    font-size: 1.25rem;
    font-weight: 500;
    margin-bottom: 1rem;
  }

  .address-block textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    margin-bottom: 1rem;
    resize: vertical;
  }

  .address-block label {
    display: flex;
    flex-direction: column;
    margin-bottom: 0.5rem;
  }

  .address-block span {
    font-size: 0.9rem;
    color: #6b7280;
    margin-bottom: 0.25rem;
  }

  .address-block input {
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
  }

  .address-block input:focus {
    outline: none;
    border-color: #2563eb;
  }

  /* Items Section */
  .items-section h2 {
    font-size: 1.25rem;
    font-weight: 500;
    margin-bottom: 1rem;
  }

  .item-form {
    display: flex;
    flex-wrap: wrap;
    align-items: flex-end;
    margin-bottom: 1rem;
  }

  .item-form input {
    flex: 1 1 20%;
    padding: 0.5rem;
    margin-right: 1rem;
    margin-bottom: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
  }

  .item-form input:focus {
    outline: none;
    border-color: #2563eb;
  }

  .add-button {
    background-color: #2563eb;
    color: #fff;
    border: none;
    padding: 0.5rem;
    border-radius: 4px;
    cursor: pointer;
  }

  .add-button:hover {
    background-color: #1d4ed8;
  }

  .items-table {
    width: 100%;
    border-collapse: collapse;
  }

  .items-table th,
  .items-table td {
    padding: 0.75rem;
    border: 1px solid #d1d5db;
    text-align: left;
  }

  .items-table th {
    background-color: #f3f4f6;
  }

  .items-table td input {
    width: 100%;
    border: none;
    padding: 0.25rem;
  }

  .items-table td input:focus {
    outline: none;
    border-bottom: 1px solid #2563eb;
  }

  .items-table .actions {
    width: 50px;
    text-align: center;
  }

  /* Totals */
  .totals {
    margin-top: 2rem;
    max-width: 400px;
    margin-left: auto;
  }

  .totals-row {
    display: flex;
    justify-content: space-between;
    padding: 0.5rem 0;
  }

  .totals-row input {
    width: 80px;
    padding: 0.25rem;
    text-align: right;
    border: 1px solid #d1d5db;
    border-radius: 4px;
  }

  .totals-row input:focus {
    outline: none;
    border-color: #2563eb;
  }

  .total-due {
    font-weight: 700;
    font-size: 1.25rem;
  }

  /* Switch */
  .switch {
    position: relative;
    display: inline-block;
    width: 36px;
    height: 18px;
    margin-left: 0.5rem;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .switch .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #d1d5db;
    transition: 0.4s;
    border-radius: 18px;
  }

  .switch .slider:before {
    position: absolute;
    content: '';
    height: 14px;
    width: 14px;
    left: 2px;
    bottom: 2px;
    background-color: white;
    transition: 0.4s;
    border-radius: 50%;
  }

  .switch input:checked + .slider {
    background-color: #2563eb;
  }

  .switch input:checked + .slider:before {
    transform: translateX(18px);
  }

  /* Payment Info */
  .payment-info {
    margin-top: 2rem;
  }

  .payment-info h2 {
    font-size: 1.25rem;
    font-weight: 500;
    margin-bottom: 1rem;
  }

  .payment-fields {
    display: flex;
    flex-wrap: wrap;
  }

  .payment-fields label {
    flex: 1 1 45%;
    display: flex;
    flex-direction: column;
    margin-bottom: 1rem;
    margin-right: 1rem;
  }

  .payment-fields span {
    font-size: 0.9rem;
    color: #6b7280;
    margin-bottom: 0.25rem;
  }

  .payment-fields input {
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 4px;
  }

  .payment-fields input:focus {
    outline: none;
    border-color: #2563eb;
  }

  /* Footer */
  .footer {
    display: flex;
    justify-content: center;
    margin-top: 2rem;
  }

  .download-button {
    background-color: #2563eb;
    color: #fff;
    border: none;
    padding: 0.75rem 1.5rem;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .download-button:hover {
    background-color: #1d4ed8;
  }

  /* Print Styles */
  @media print {
    body {
      background-color: #fff;
    }

    .container {
      border: none;
      padding: 0;
    }

    .header,
    .addresses,
    .items-section,
    .totals,
    .payment-info,
    .footer {
      margin: 0;
    }

    .icon-button,
    .add-button,
    .download-button,
    .switch {
      display: none;
    }

    input,
    textarea {
      border: none;
    }

    .items-table th,
    .items-table td {
      border: 1px solid #000;
    }
  }
</style>
