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
    // Add more currencies as needed
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

  let isDiscountEnabled = appState.discountPercent !== '';
  let isTaxEnabled = appState.taxPercent !== '';
</script>

<svelte:body on:click={() => save()} />

<div class="invoice-container">
  <!-- Header -->
  <header class="invoice-header">
    <div class="company-info">
      {#if appState.company.logo}
        <div class="logo-container">
          <img src={appState.company.logo} alt="Company Logo" class="company-logo" />
          <button class="icon-button" on:click={() => { appState.company.logo = ''; save(); }}>
            <Trash2 />
          </button>
        </div>
      {:else}
        <label class="logo-upload">
          <input type="file" accept=".png,.jpg,.jpeg,.webp" on:change={handleImageChange} />
          <ImagePlus />
          <span>Upload Logo</span>
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
      <div class="details-grid">
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
      <h2>From</h2>
      <textarea
        bind:value={appState.invoice.from}
        placeholder="Your Address"
        rows="4"
      ></textarea>
      <div class="contact-info">
        <div class="input-group">
          <label>{appState.invoice.fromTaxIdName}</label>
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
      <h2>To</h2>
      <textarea
        bind:value={appState.invoice.to}
        placeholder="Client's Address"
        rows="4"
      ></textarea>
      <div class="contact-info">
        <div class="input-group">
          <label>{appState.invoice.toTaxIdName}</label>
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
        placeholder="Description"
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
      <button type="submit" class="add-button">
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
            <button class="icon-button" on:click={() => deleteItem(index)}>
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
      <span>{getCurrencySymbol(appState.currency)}{subTotal}</span>
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
      <span>{getCurrencySymbol(appState.currency)}{taxableAmount}</span>
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
      <span>{getCurrencySymbol(appState.currency)}{totalDue}</span>
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
    background-color: #f9fafb;
    color: #111827;
  }

  .invoice-container {
    max-width: 800px;
    margin: 40px auto;
    background-color: #ffffff;
    border-radius: 8px;
    padding: 40px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  }

  h1, h2 {
    margin-bottom: 16px;
    font-weight: 600;
  }

  h1 {
    font-size: 2rem;
  }

  h2 {
    font-size: 1.25rem;
  }

  label {
    display: block;
    font-size: 0.875rem;
    color: #6b7280;
    margin-bottom: 4px;
  }

  input, textarea, select {
    width: 100%;
    padding: 12px;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    font-size: 1rem;
    color: #111827;
    background-color: #f9fafb;
    transition: border-color 0.2s;
  }

  input:focus, textarea:focus, select:focus {
    outline: none;
    border-color: #3b82f6;
    background-color: #ffffff;
  }

  .icon-button {
    background: none;
    border: none;
    cursor: pointer;
    color: #6b7280;
    transition: color 0.2s;
  }

  .icon-button:hover {
    color: #111827;
  }

  /* Header */
  .invoice-header {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
  }

  .company-info {
    flex: 1 1 40%;
  }

  .company-name {
    font-size: 1.5rem;
    font-weight: 700;
    border: none;
    border-bottom: 1px solid #d1d5db;
    margin-bottom: 24px;
    padding: 8px 0;
  }

  .company-logo {
    max-height: 80px;
    margin-bottom: 24px;
  }

  .logo-container {
    position: relative;
  }

  .logo-container .icon-button {
    position: absolute;
    top: 0;
    right: 0;
  }

  .logo-upload {
    display: flex;
    align-items: center;
    cursor: pointer;
    color: #6b7280;
    margin-bottom: 24px;
  }

  .logo-upload input {
    display: none;
  }

  .logo-upload:hover {
    color: #111827;
  }

  .invoice-details {
    flex: 1 1 50%;
    text-align: right;
  }

  .details-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
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

  .contact-info {
    margin-top: 16px;
  }

  .contact-info .input-group {
    margin-bottom: 16px;
  }

  /* Items */
  .items-section {
    margin-top: 40px;
  }

  .item-form {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
    margin-bottom: 24px;
  }

  .item-form input {
    flex: 1 1 30%;
  }

  .add-button {
    background: #3b82f6;
    color: #ffffff;
    border: none;
    padding: 12px 16px;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    transition: background 0.2s;
  }

  .add-button:hover {
    background: #2563eb;
  }

  .items-table {
    width: 100%;
    border-collapse: collapse;
  }

  .table-header, .table-row {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr 1fr 40px;
    gap: 16px;
    align-items: center;
  }

  .table-header {
    border-bottom: 2px solid #d1d5db;
    padding-bottom: 8px;
    margin-bottom: 16px;
    color: #6b7280;
  }

  .table-row {
    margin-bottom: 16px;
  }

  .table-row input {
    background-color: #f9fafb;
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
    align-items: center;
    margin-bottom: 16px;
    font-size: 1rem;
  }

  .totals-row input {
    width: 100px;
    text-align: right;
  }

  .total-due {
    font-weight: 700;
    font-size: 1.25rem;
  }

  .switch {
    position: relative;
    display: inline-block;
    width: 40px;
    height: 24px;
    margin-left: 8px;
  }

  .switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .switch .slider {
    position: absolute;
    cursor: pointer;
    background-color: #d1d5db;
    border-radius: 24px;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    transition: background-color 0.2s;
  }

  .switch .slider:before {
    position: absolute;
    content: '';
    height: 20px;
    width: 20px;
    left: 2px;
    bottom: 2px;
    background-color: #ffffff;
    border-radius: 50%;
    transition: transform 0.2s;
  }

  .switch input:checked + .slider {
    background-color: #3b82f6;
  }

  .switch input:checked + .slider:before {
    transform: translateX(16px);
  }

  /* Payment Info */
  .payment-info {
    margin-top: 40px;
  }

  .payment-fields {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 16px;
  }

  /* Footer */
  .invoice-footer {
    text-align: center;
    margin-top: 40px;
  }

  .download-button {
    background: #3b82f6;
    color: #ffffff;
    border: none;
    padding: 12px 24px;
    font-size: 1rem;
    border-radius: 4px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 8px;
    transition: background 0.2s;
  }

  .download-button:hover {
    background: #2563eb;
  }

  /* Responsive */
  @media (max-width: 768px) {
    .details-grid {
      grid-template-columns: 1fr;
    }

    .addresses {
      flex-direction: column;
    }

    .address-block {
      margin-right: 0;
    }

    .item-form input {
      flex: 1 1 100%;
    }

    .table-header, .table-row {
      grid-template-columns: 1fr;
    }

    .table-header div, .table-row div {
      margin-bottom: 8px;
    }

    .totals-section {
      margin-left: 0;
    }
  }

  /* Print Styles */
  @media print {
    .invoice-container {
      box-shadow: none;
      border: none;
      margin: 0;
      padding: 0;
    }

    .add-button, .icon-button, .download-button, .switch {
      display: none;
    }

    input, textarea, select {
      border: none;
    }
  }
</style>
