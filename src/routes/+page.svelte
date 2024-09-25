<script>
  import { Plus, Trash, ImagePlus, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: 'Wayne Enterprises', logo: '' },
    invoice: {
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' },
    },
    items: [
      { desc: 'SEO Consultation', price: '5000', quantity: '1' },
      { desc: 'Social Media Strategy', price: '2000', quantity: '3' },
      { desc: 'Content Creation', price: '10000', quantity: '1' },
    ],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: '',
    },
    currency: 'EUR',
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    { code: 'GBP', symbol: '£', name: 'British Pound', flag: '🇬🇧' },
  ];

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
    if (itemDesc && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        { desc: itemDesc, price: parseFloat(itemPrice).toFixed(2), quantity: parseFloat(itemQty).toFixed(2) },
      ];
      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;
      save();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxId: '',
        toContact: { mail: '', phone: '' },
      },
      items: [
        { desc: 'SEO Consultation', price: '5000', quantity: '1' },
        { desc: 'Social Media Strategy', price: '2000', quantity: '3' },
        { desc: 'Content Creation', price: '10000', quantity: '1' },
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: '',
      },
      currency: 'EUR',
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

  $: subTotal = appState.items.reduce((total, item) => total + item.price * item.quantity, 0).toFixed(2);
  $: discountAmount = (subTotal * (+appState.discountPercent || 0) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = (taxableAmount * (+appState.taxPercent || 0) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
  <meta name="description" content="Create and download invoices instantly, with zero signups or tracking." />
</svelte:head>

<div class="container">
  <!-- Header -->
  <div class="header">
    <div class="logo-section">
      <button on:click={() => document.getElementById('imageInput').click()} class="logo-btn">
        <ImagePlus />
        <input id="imageInput" type="file" accept=".png,.jpg,.jpeg,.webp" on:change={handleImageChange} class="hidden" />
      </button>
      {#if appState.company.logo}
        <img src={appState.company.logo} alt="Company Logo" class="company-logo" />
      {/if}
      <input class="company-name" type="text" bind:value={appState.company.name} />
    </div>
    <div class="invoice-details">
      <h2 class="invoice-title">
        INVOICE : <input type="text" size="4" placeholder="#0001" maxlength="5" bind:value={appState.invoice.number} />
      </h2>
      <button on:click={() => reset()} class="reset-btn">Reset</button>
      <div class="date-currency">
        <input type="date" bind:value={appState.invoice.created} placeholder="Created : dd-mm-yyyy" class="date-input" />
        <input type="date" bind:value={appState.invoice.due} placeholder="Due : dd-mm-yyyy" class="date-input" />
        <select bind:value={appState.currency} class="currency-select">
          {#each currencyOptions as option}
            <option value={option.code}>{option.flag} {option.code} - {option.name}</option>
          {/each}
        </select>
      </div>
    </div>
  </div>

  <!-- Contact Info -->
  <h3 class="section-title">Contact Info</h3>
  <div class="contact-info">
    <textarea placeholder="Enter sender's name and address" bind:value={appState.invoice.from} class="contact-input"></textarea>
    <textarea placeholder="Enter recipient's name and address" bind:value={appState.invoice.to} class="contact-input"></textarea>
    <input type="text" placeholder="Enter Tax Name" bind:value={appState.invoice.fromTaxId} class="contact-input" />
    <input type="text" placeholder="Enter Tax Name" bind:value={appState.invoice.toTaxId} class="contact-input" />
    <input type="email" placeholder="Enter Email" bind:value={appState.invoice.fromContact.mail} class="contact-input" />
    <input type="email" placeholder="Enter Email" bind:value={appState.invoice.toContact.mail} class="contact-input" />
    <input type="text" placeholder="Enter Phone" bind:value={appState.invoice.fromContact.phone} class="contact-input" />
    <input type="text" placeholder="Enter Phone" bind:value={appState.invoice.toContact.phone} class="contact-input" />
  </div>

  <!-- Item Descriptions -->
  <h3 class="section-title">Item Descriptions</h3>
  <form class="item-form">
    <input id="descBox" type="text" bind:value={itemDesc} placeholder="Item name" class="item-input" />
    <input type="number" bind:value={itemPrice} placeholder="Unit Price" min="0" step="0.01" class="item-input" />
    <input type="number" bind:value={itemQty} placeholder="Quantity" min="1" step="1" class="item-input" />
    <button on:click={() => addItem()} class="add-btn"><Plus /></button>
  </form>

  <!-- Items Table -->
  <div class="item-description">
    <div class="item-header">
      <p>Item name</p>
      <p>Unit Price</p>
      <p>Qty</p>
      <p>Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="item-row">
        <button on:click={() => deleteItem(index)} class="delete-btn"><Trash /></button>
        <p contenteditable="true">{item.desc}</p>
        <input type="text" bind:value={item.price} class="item-input" />
        <input type="text" bind:value={item.quantity} class="item-input" />
        <p>{(item.price * item.quantity).toFixed(2)}</p>
      </div>
    {/each}
    <div class="totals">
      <div class="toggle-section">
        <p>Discount %</p>
        <button class="{isDiscountEnabled ? 'toggle active' : 'toggle'}" on:click={() => (isDiscountEnabled = !isDiscountEnabled)}></button>
        <input type="text" bind:value={appState.discountPercent} disabled={!isDiscountEnabled} />
      </div>
      <div class="toggle-section">
        <p>Tax %</p>
        <button class="{isTaxEnabled ? 'toggle active' : 'toggle'}" on:click={() => (isTaxEnabled = !isTaxEnabled)}></button>
        <input type="text" bind:value={appState.taxPercent} disabled={!isTaxEnabled} />
      </div>
      <div class="total-due">
        <p>Total Due</p>
        <p>{totalDue}</p>
      </div>
    </div>
  </div>

  <!-- Payment Info -->
  <h3 class="section-title">Payment Info</h3>
  <div class="payment-info">
    <input type="text" placeholder="Account No" bind:value={appState.payment.accountNumber} class="payment-input" />
    <input type="text" placeholder="Account Name" bind:value={appState.payment.accountName} class="payment-input" />
    <input type="text" placeholder="Bank Name" bind:value={appState.payment.bank} class="payment-input" />
    <input type="text" placeholder="IFSC" bind:value={appState.payment.ifsc} class="payment-input" />
    <input type="text" placeholder="SWIFT Code" bind:value={appState.payment.swiftCode} class="payment-input" />
  </div>

  <!-- Download Button -->
  <div class="download-section">
    <button on:click={() => window.print()} class="download-btn">
      <Download />
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    padding: 20px;
    max-width: 100%;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 10px;
  }

  .contact-info {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
    margin-bottom: 20px;
  }

  .item-description {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
    background-color: #f7f7f7;
    padding: 10px;
    border-radius: 8px;
  }

  .payment-info {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
    margin-bottom: 20px;
  }

  /* Responsive styling */
  @media (max-width: 768px) {
    .header {
      flex-direction: column;
      align-items: flex-start;
    }

    .contact-info,
    .payment-info {
      grid-template-columns: 1fr;
    }

    .item-description {
      grid-template-columns: 1fr;
    }
  }

  @media (max-width: 480px) {
    .header {
      gap: 5px;
    }

    .contact-info input,
    .payment-info input,
    .item-description input {
      width: 100%;
    }

    .item-description {
      padding: 5px;
    }
  }
</style>
