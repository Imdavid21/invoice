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
      fromTaxIdName: 'Tax ID',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: 'Tax ID',
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
    // ... add other currencies as needed
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && ['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
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
      company: { name: 'Your Company', logo: '' },
      invoice: {
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Tax ID',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Tax ID',
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

<div class="invoice-container">
  <!-- Header -->
  <header class="invoice-header">
    <div class="company-section">
      {#if appState.company.logo}
        <div class="logo-wrapper">
          <img src={appState.company.logo} alt="Company Logo" class="company-logo" />
          <button class="remove-logo" on:click={() => { appState.company.logo = ''; save(); }}>
            <Trash2 />
          </button>
        </div>
      {:else}
        <label class="logo-upload">
          <ImagePlus />
          <span>Upload Logo</span>
          <input type="file" accept=".png,.jpg,.jpeg,.webp" on:change={handleImageChange} />
        </label>
      {/if}
      <input
        type="text"
        bind:value={appState.company.name}
        placeholder="Company Name"
        class="company-name"
      />
    </div>
    <div class="invoice-details">
      <h1>Invoice</h1>
      <div class="invoice-info">
        <div class="input-group">
          <label>Invoice Number</label>
          <input type="text" bind:value={appState.invoice.number} placeholder="#0001" />
        </div>
        <div class="input-group">
          <label>Invoice Date</label>
          <input type="date" bind:value={appState.invoice.created} />
        </div>
        <div class="input-group">
          <label>Due Date</label>
          <input type="date" bind:value={appState.invoice.due} />
        </div>
        <div class="input-group">
          <label>Currency</label>
          <select bind:value={appState.currency}>
            {#each currencyOptions as option}
              <option value={option.code}>{option.flag} {option.code} - {option.name}</option>
            {/each}
          </select>
        </div>
      </div>
    </div>
  </header>

  <hr />

  <!-- Addresses -->
  <section class="addresses">
    <div class="address-block">
      <h2>Bill From</h2>
      <textarea
        bind:value={appState.invoice.from}
        placeholder="Your Address"
        rows="4"
      ></textarea>
      <div class="contact-info">
        <div class="input-group">
          <label>{appState.invoice.fromTaxIdName || 'Tax ID'}</label>
          <input type="text" bind:value={appState.invoice.fromTaxId} />
        </div>
        <div class="input-group">
          <label>Email</label>
          <input type="email" bind:value={appState.invoice.fromContact.mail} />
        </div>
        <div class="input-group">
          <label>Phone</label>
          <input type="tel" bind:value={appState.invoice.fromContact.phone} />
        </div>
      </div>
    </div>
    <div class="address-block">
      <h2>Bill To</h2>
      <textarea
        bind:value={appState.invoice.to}
        placeholder="Client's Address"
        rows="4"
      ></textarea>
      <div class="contact-info">
        <div class="input-group">
          <label>{appState.invoice.toTaxIdName || 'Tax ID'}</label>
          <input type="text" bind:value={appState.invoice.toTaxId} />
        </div>
        <div class="input-group">
          <label>Email</label>
          <input type="email" bind:value={appState.invoice.toContact.mail} />
        </div>
        <div class="input-group">
          <label>Phone</label>
          <input type="tel" bind:value={appState.invoice.toContact.phone} />
        </div>
      </div>
    </div>
  </section>

  <hr />

  <!-- Items -->
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
      <button type="submit" class="add-item">
        <Plus />
      </button>
    </form>

    <div class="items-table">
      <div class="table-header">
        <div>Description</div>
        <div>Unit Price</div>
        <div>Quantity</div>
        <div>Total</div>
        <div></div>
      </div>
      {#each appState.items as item, index}
        <div class="table-row">
          <div>
            <input
              type="text"
              bind:value={item.desc}
              on:input={() => save()}
              required
            />
          </div>
          <div>
            <input
              type="number"
              bind:value={item.price}
              on:input={() => save()}
              min="0"
              step="0.01"
              required
            />
          </div>
          <div>
            <input
              type="number"
              bind:value={item.quantity}
              on:input={() => save()}
              min="1"
              step="1"
              required
            />
          </div>
          <div>{(item.price * item.quantity).toFixed(2)}</div>
          <div>
            <button class="delete-item" on:click={() => deleteItem(index)}>
              <Trash2 />
            </button>
          </div>
        </div>
      {/each}
    </div>
  </section>

  <!-- Totals -->
  <section class="totals-section">
    <div class="totals-row">
      <span>Subtotal</span>
      <span>{subTotal}</span>
    </div>
    <div class="totals-row">
      <span>
        Discount (%)
        <label class="toggle-switch">
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
        <label class="toggle-switch">
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

  <!-- Payment Info -->
  <section class="payment-info">
    <h2>Payment Information</h2>
    <div class="payment-fields">
      <div class="input-group">
        <label>Account Number</label>
        <input type="text" bind:value={appState.payment.accountNumber} maxlength="20" />
      </div>
      <div class="input-group">
        <label>Account Name</label>
        <input type="text" bind:value={appState.payment.accountName} maxlength="28" />
      </div>
      <div class="input-group">
        <label>Bank Name</label>
        <input type="text" bind:value={appState.payment.bank} maxlength="28" />
      </div>
      <div class="input-group">
        <label>IFSC</label>
        <input type="text" bind:value={appState.payment.ifsc} maxlength="11" />
      </div>
      <div class="input-group">
        <label>SWIFT Code</label>
        <input type="text" bind:value={appState.payment.swiftCode} maxlength="11" />
      </div>
    </div>
  </section>

  <!-- Download Button -->
  <footer class="invoice-footer">
    <button class="download-button" on:click={() => window.print()}>
      <Download />
      <span>Download Invoice</span>
    </button>
  </footer>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap');

  /* Global Styles */
  body {
    margin: 0;
    font-family: 'Inter', sans-serif;
    background-color: #fafafa;
    color: #333;
  }

  .invoice-container {
    max-width: 900px;
    margin: 40px auto;
    background-color: #fff;
    border-radius: 8px;
    padding: 40px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }

  /* Header */
  .invoice-header {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: flex-start;
  }

  .company-section {
    flex: 1 1 40%;
  }

  .logo-wrapper {
    position: relative;
  }

  .company-logo {
    max-height: 80px;
    margin-bottom: 20px;
  }

  .remove-logo {
    position: absolute;
    top: 0;
    right: 0;
    background: #fff;
    border: none;
    cursor: pointer;
    padding: 4px;
    border-radius: 50%;
    transition: background 0.2s;
  }

  .remove-logo:hover {
    background: #f0f0f0;
  }

  .logo-upload {
    display: flex;
    align-items: center;
    cursor: pointer;
    color: #666;
    margin-bottom: 20px;
  }

  .logo-upload input {
    display: none;
  }

  .logo-upload:hover {
    color: #000;
  }

  .company-name {
    font-size: 1.5rem;
    font-weight: 700;
    border: none;
    border-bottom: 1px solid #ccc;
    padding: 8px 0;
    width: 100%;
    transition: border-color 0.2s;
  }

  .company-name:focus {
    outline: none;
    border-color: #000;
  }

  .invoice-details {
    flex: 1 1 50%;
    text-align: right;
  }

  .invoice-details h1 {
    font-size: 2rem;
    font-weight: 700;
    margin-bottom: 20px;
  }

  .invoice-info {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-end;
  }

  .invoice-info .input-group {
    width: 48%;
    margin-bottom: 10px;
    margin-left: 4%;
  }

  .invoice-info .input-group:nth-child(odd) {
    margin-left: 0;
  }

  .invoice-info label {
    font-size: 0.9rem;
    color: #666;
    margin-bottom: 4px;
  }

  .invoice-info input,
  .invoice-info select {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.95rem;
    transition: border-color 0.2s;
  }

  .invoice-info input:focus,
  .invoice-info select:focus {
    outline: none;
    border-color: #000;
  }

  /* Addresses */
  .addresses {
    display: flex;
    flex-wrap: wrap;
    margin-top: 40px;
  }

  .address-block {
    flex: 1 1 45%;
    margin-right: 5%;
    margin-bottom: 40px;
  }

  .address-block:last-child {
    margin-right: 0;
  }

  .address-block h2 {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 20px;
  }

  .address-block textarea {
    width: 100%;
    padding: 12px;
    border: 1px solid #ccc;
    border-radius: 4px;
    resize: vertical;
    font-size: 0.95rem;
    transition: border-color 0.2s;
    margin-bottom: 20px;
  }

  .address-block textarea:focus {
    outline: none;
    border-color: #000;
  }

  .contact-info {
    display: flex;
    flex-wrap: wrap;
  }

  .contact-info .input-group {
    width: 48%;
    margin-right: 4%;
    margin-bottom: 10px;
  }

  .contact-info .input-group:last-child {
    margin-right: 0;
  }

  .contact-info label {
    font-size: 0.9rem;
    color: #666;
    margin-bottom: 4px;
  }

  .contact-info input {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.95rem;
    transition: border-color 0.2s;
  }

  .contact-info input:focus {
    outline: none;
    border-color: #000;
  }

  /* Items */
  .items-section {
    margin-top: 40px;
  }

  .items-section h2 {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 20px;
  }

  .item-form {
    display: flex;
    flex-wrap: wrap;
    margin-bottom: 20px;
  }

  .item-form input {
    flex: 1 1 23%;
    margin-right: 2%;
    padding: 8px;
    font-size: 0.95rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    transition: border-color 0.2s;
    margin-bottom: 10px;
  }

  .item-form input:focus {
    outline: none;
    border-color: #000;
  }

  .item-form input:last-child {
    margin-right: 0;
  }

  .add-item {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 1.5rem;
    color: #666;
    transition: color 0.2s;
    padding: 0;
    margin-left: 10px;
  }

  .add-item:hover {
    color: #000;
  }

  .items-table {
    border-collapse: collapse;
    width: 100%;
  }

  .items-table .table-header,
  .items-table .table-row {
    display: flex;
    border-bottom: 1px solid #e0e0e0;
  }

  .items-table .table-header div,
  .items-table .table-row div {
    flex: 1 1 20%;
    padding: 12px;
    font-size: 0.95rem;
  }

  .items-table .table-header div {
    font-weight: 600;
    color: #666;
  }

  .items-table .table-row div {
    align-items: center;
  }

  .items-table .table-row div input {
    width: 100%;
    border: none;
    padding: 8px;
    font-size: 0.95rem;
    transition: border-color 0.2s;
  }

  .items-table .table-row div input:focus {
    outline: none;
    border-bottom: 1px solid #000;
  }

  .delete-item {
    background: none;
    border: none;
    cursor: pointer;
    color: #666;
    transition: color 0.2s;
    padding: 0;
  }

  .delete-item:hover {
    color: #000;
  }

  /* Totals */
  .totals-section {
    margin-top: 40px;
    max-width: 400px;
    margin-left: auto;
  }

  .totals-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 10px;
    font-size: 0.95rem;
  }

  .totals-row input {
    width: 80px;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    text-align: right;
    transition: border-color 0.2s;
  }

  .totals-row input:focus {
    outline: none;
    border-color: #000;
  }

  .totals-row.total-due {
    font-weight: 700;
    font-size: 1.1rem;
  }

  .toggle-switch {
    position: relative;
    display: inline-block;
    width: 36px;
    height: 20px;
    margin-left: 8px;
  }

  .toggle-switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .toggle-switch .slider {
    position: absolute;
    cursor: pointer;
    background-color: #ccc;
    border-radius: 20px;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    transition: background-color 0.2s;
  }

  .toggle-switch .slider:before {
    position: absolute;
    content: '';
    height: 16px;
    width: 16px;
    left: 2px;
    bottom: 2px;
    background-color: #fff;
    border-radius: 50%;
    transition: transform 0.2s;
  }

  .toggle-switch input:checked + .slider {
    background-color: #000;
  }

  .toggle-switch input:checked + .slider:before {
    transform: translateX(16px);
  }

  /* Payment Info */
  .payment-info {
    margin-top: 40px;
  }

  .payment-info h2 {
    font-size: 1.25rem;
    font-weight: 600;
    margin-bottom: 20px;
  }

  .payment-fields {
    display: flex;
    flex-wrap: wrap;
  }

  .payment-fields .input-group {
    flex: 1 1 48%;
    margin-right: 4%;
    margin-bottom: 20px;
  }

  .payment-fields .input-group:nth-child(2n) {
    margin-right: 0;
  }

  .payment-fields label {
    font-size: 0.9rem;
    color: #666;
    margin-bottom: 4px;
  }

  .payment-fields input {
    width: 100%;
    padding: 8px;
    font-size: 0.95rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    transition: border-color 0.2s;
  }

  .payment-fields input:focus {
    outline: none;
    border-color: #000;
  }

  /* Footer */
  .invoice-footer {
    text-align: center;
    margin-top: 40px;
  }

  .download-button {
    background: #000;
    color: #fff;
    border: none;
    padding: 12px 24px;
    font-size: 1rem;
    border-radius: 4px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    transition: background 0.2s;
  }

  .download-button:hover {
    background: #333;
  }

  .download-button svg {
    margin-right: 8px;
  }

  /* Responsive */
  @media (max-width: 768px) {
    .invoice-info .input-group,
    .contact-info .input-group,
    .payment-fields .input-group {
      width: 100%;
      margin-right: 0;
    }

    .items-table .table-header div,
    .items-table .table-row div {
      flex: 1 1 100%;
    }

    .items-table .table-row {
      flex-wrap: wrap;
    }

    .items-table .table-row div {
      margin-bottom: 10px;
    }

    .items-table .table-row div:last-child {
      margin-bottom: 0;
    }
  }

  /* Print Styles */
  @media print {
    .invoice-container {
      box-shadow: none;
      border: none;
      padding: 0;
    }

    .remove-logo,
    .add-item,
    .delete-item,
    .download-button {
      display: none;
    }

    input,
    textarea,
    select {
      border: none;
    }

    .toggle-switch {
      display: none;
    }
  }
</style>
