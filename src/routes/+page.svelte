<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '001',
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
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '',
    discountPercent: '',
    note: '',
    paymentFields: [
      { label: 'Account No.', value: '' },
      { label: 'Account Name', value: '' },
      { label: 'Bank Name', value: '' },
      { label: 'IFSC', value: '' },
      { label: 'SWIFT Code', value: '' }
    ],
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar' },
    { code: 'EUR', symbol: '€', name: 'Euro' },
    { code: 'GBP', symbol: '£', name: 'British Pound' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar' }
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (file && (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')) {
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
    // Intentionally left blank to prevent localStorage usage
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc,
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2)
        }
      ];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
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
        number: '007',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      paymentFields: [
        { label: 'Account No.', value: '' },
        { label: 'Account Name', value: '' },
        { label: 'Bank Name', value: '' },
        { label: 'IFSC', value: '' },
        { label: 'SWIFT Code', value: '' }
      ],
      currency: 'USD'
    };

    save();
  }

  function addPaymentField() {
    appState.paymentFields.push({ label: 'New Field', value: '' });
    save();
  }

  function removePaymentField(index) {
    appState.paymentFields.splice(index, 1);
    save();
  }

  onMount(() => {
    reset();
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

  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<div
  class="max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-montserrat bg-[#FAFAFA] border border-[#E2E2E2] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- From & To Section with Responsive Inputs -->
  <div class="flex flex-row gap-6">
    <div class="flex-1">
      <h4 class="mb-2 font-bold">From</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Rick Sanchez, Earth Dimension C-137"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
      ></textarea>
      <div class="flex flex-col gap-1 mt-2">
        <label class="text-sm">Tax Name:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromTaxIdName}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="Intergalactic Federation ID"
        />
        <label class="text-sm">Tax ID:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromTaxId}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="C137-TAX-99"
        />
        <label class="text-sm">Email:</label>
        <input
          type="email"
          bind:value={appState.invoice.fromContact.mail}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="rick@sanchezlabs.com"
        />
        <label class="text-sm">Phone:</label>
        <input
          type="text"
          bind:value={appState.invoice.fromContact.phone}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="+1-800-WUBBA"
        />
      </div>
    </div>

    <div class="flex-1">
      <h4 class="mb-2 font-bold">To</h4>
      <textarea
        style="resize: none; padding: 8px 12px;"
        placeholder="Morty Smith, Earth Dimension C-137"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
      ></textarea>
      <div class="flex flex-col gap-1 mt-2">
        <label class="text-sm">Tax Name:</label>
        <input
          type="text"
          bind:value={appState.invoice.toTaxIdName}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="Galactic Taxpayer ID"
        />
        <label class="text-sm">Tax ID:</label>
        <input
          type="text"
          bind:value={appState.invoice.toTaxId}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="MORTY-C137-TAX"
        />
        <label class="text-sm">Email:</label>
        <input
          type="email"
          bind:value={appState.invoice.toContact.mail}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="morty@smiths.com"
        />
        <label class="text-sm">Phone:</label>
        <input
          type="text"
          bind:value={appState.invoice.toContact.phone}
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] focus:ring-2 focus:ring-blue-300 transition-all duration-200"
          placeholder="+1-800-AW-GEEZ"
        />
      </div>
    </div>
  </div>
</div>
